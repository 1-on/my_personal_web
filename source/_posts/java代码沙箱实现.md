---
title: java实现代码沙箱
abbrlink: 7
categories: java学习笔记
tags: java项目
updated: 2023-01-17
date: 2023-01-17

---





### 流程

1. 将用户代码保存为文件
2. 编译代码，得到class文件
3. 执行代码
4. 收集整理输出结果
5. 文件清理

```java
public ExecuteCodeResponse executeCode(ExecuteCodeRequest executeCodeRequest) {
//        System.setSecurityManager(new DefaultSecurityManager());
        List<String> inputList = executeCodeRequest.getInputList();
        String code = executeCodeRequest.getCode();
        String language = executeCodeRequest.getLanguage();
        // 1)把用户代码保存为文件
        File userCodeFile = saveCodeToFile(code);
        // 2)编译代码，得到 class 文件
        ExecuteMessage compileFileExecuteMessage = compileFile(userCodeFile);
        System.out.println(compileFileExecuteMessage);
        // 编译错误直接返回
        if (compileFileExecuteMessage.getExitValue() != 0) {
            ExecuteCodeResponse compilerErrorResponse = new ExecuteCodeResponse();
            compilerErrorResponse.setStatus(3);
            JudgeInfo judgeInfo = new JudgeInfo();
            judgeInfo.setMessage("编译错误");
            compilerErrorResponse.setJudgeInfo(judgeInfo);
            return compilerErrorResponse;
        }
        // 3)执行代码
        List<ExecuteMessage> executeMessageList = runFile(userCodeFile, inputList);
        // 4)收集整理输出结果
        ExecuteCodeResponse outputResponse = getOutputResponse(executeMessageList);
        // 5)文件清理
        boolean b = clearFile(userCodeFile);
        if (!b) {
            log.error("删除文件失败，文件地址 = {}", userCodeFile.getAbsoluteFile());
        }
        return outputResponse;
    }
```

<!-- more -->

### 1.把用户代码保存为文件

```java
    public File saveCodeToFile(String code) {
        String userDir = System.getProperty("user.dir");
        String globalCodePathName = userDir + File.separator + GLOBAL_CODE_DIR_NAME;
        // 判断全局代码目录是否存在
        if (!FileUtil.exist(globalCodePathName)) {
            FileUtil.mkdir(globalCodePathName);
        }
        // 1)把用户代码隔离存放
        String userCodeParentPath = globalCodePathName + File.separator + UUID.randomUUID();
        String userCodePath = userCodeParentPath + File.separator + GLOBAL_JAVA_CLASS_NAME;
        File userCodeFile = FileUtil.writeString(code, userCodePath, StandardCharsets.UTF_8);
        System.out.println(userCodeFile);
        return userCodeFile;
    }
```

### 2.编译代码得到class文件

```java
    public ExecuteMessage compileFile(File userCodeFile) {
        String compileCmd = String.format("javac -encoding utf-8 %s", userCodeFile.getAbsoluteFile());
        try {
            Process complieProcess = Runtime.getRuntime().exec(compileCmd);
            ExecuteMessage executeMessage = ProcessUtils.runProcessAdnGetMessage(complieProcess, "编译");
            // 编译错误
            if (executeMessage.getExitValue() != 0) {
                ExecuteMessage message = new ExecuteMessage();
                message.setExitValue(executeMessage.getExitValue());
                message.setErrorMessage("编译错误");
                return message;
            }
            return executeMessage;
        } catch (Exception e) {
//            return getErrorResponse(e);
            throw new RuntimeException(e);
        }
    }
```

### 3. 执行文件、获取执行结果列表

```java
    public List<ExecuteMessage> runFile(File userCodeFile, List<String> inputList) {
        String userCodeParentPath = userCodeFile.getParentFile().getAbsolutePath();
        List<ExecuteMessage> executeMessageList = new ArrayList<>();
        for (String inputArgs : inputList) {
            String runCmd = String.format("java -Xmx256m -Dfile.encoding=UTF-8 -cp %s Main %s", userCodeParentPath, inputArgs);
            System.out.println(runCmd);
            try {
                Process runProcess = Runtime.getRuntime().exec(runCmd);
                // 超时控制
                new Thread(() -> {
                    try {
                        Thread.sleep(TIME_OUT);
                        System.out.println("超时，中断");
                        runProcess.destroy();
                    } catch (InterruptedException e) {
                        throw new RuntimeException(e);
                    }
                }).start();
                ExecuteMessage executeMessage = ProcessUtils.runProcessAdnGetMessage(runProcess, "运行");
//                ExecuteMessage executeMessage = ProcessUtils.runInteractProcessAdnGetMessage(runProcess, "运行", inputArgs);
                System.out.println(executeMessage);
                executeMessageList.add(executeMessage);
            } catch (Exception e) {
                throw new RuntimeException("执行cuowu");
            }
        }
        return executeMessageList;
    }
```

### 4.获取输出结果

```java
    public ExecuteCodeResponse getOutputResponse(List<ExecuteMessage> executeMessageList) {
        ExecuteCodeResponse executeCodeResponse = new ExecuteCodeResponse();
        List<String> outputList = new ArrayList<>();
        // 取用时最大值，判断是否超时
        long maxTime = 0;
        for (ExecuteMessage executeMessage : executeMessageList) {
            String errorMessage = executeMessage.getErrorMessage();
            // 存在错误
            if (StrUtil.isNotBlank(errorMessage)) {
                {
                    executeCodeResponse.setMessage(errorMessage);
                    // 用户提交代码执行错误
                    executeCodeResponse.setStatus(3);
                }
            }
            outputList.add(executeMessage.getMessage());
            Long time = executeMessage.getTime();
            if (time != null) {
                maxTime = Math.max(maxTime, time);
            }
        }
        // 正常运行完成
        if (outputList.size() == executeMessageList.size()) {
            executeCodeResponse.setStatus(1);
        }
        executeCodeResponse.setOutputList(outputList);
        JudgeInfo judgeInfo = new JudgeInfo();
        judgeInfo.setTime(maxTime);
//        judgeInfo.setMemoryLimit(0L);
        // 要用第三方库，很麻烦 此处不调用
//        judgeInfo.setMemoryLimit();
        executeCodeResponse.setJudgeInfo(judgeInfo);
        return executeCodeResponse;
    }
```

### 5.删除文件

```java
    public boolean clearFile(File userCodeFile) {
        if (userCodeFile.getParentFile() != null) {
            String userCodeParentPath = userCodeFile.getParentFile().getAbsolutePath();
            boolean del = FileUtil.del(userCodeParentPath);
            System.out.println("删除" + (del ? "成功" : "失败"));
            return del;
        }
        return true;
    }
```

### 进程工具类

```java
ublic class ProcessUtils {
    /**
     * 执行进程并获取信息
     *
     * @param runProcess 进程
     * @param opName     操作名称
     * @return
     */
    public static ExecuteMessage runProcessAdnGetMessage(Process runProcess, String opName) {
        ExecuteMessage executeMessage = new ExecuteMessage();
        try {
            StopWatch stopWatch = new StopWatch();
            stopWatch.start();
            int exitValue = runProcess.waitFor();
            executeMessage.setExitValue(exitValue);
            // 正常退出
            if (exitValue == 0) {
                System.out.println(opName + "成功");
                // 分批获取进程的输出
                BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(runProcess.getInputStream()));
                List<String> outputList = new ArrayList<>();
                // 逐行读取
                String compileOutputLine;
                while ((compileOutputLine = bufferedReader.readLine()) != null) {
                    outputList.add(compileOutputLine);
                }
                executeMessage.setMessage(StringUtils.join(outputList, "\n"));
            } else {
                // 异常退出
                System.out.println(opName + "失败" + exitValue);

                // 分批获取进程的正常输出
                BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(runProcess.getInputStream()));

                List<String> outputList = new ArrayList<>();
                // 逐行读取
                String compileOutputLine;
                while ((compileOutputLine = bufferedReader.readLine()) != null) {
                    outputList.add(compileOutputLine);
                }
                executeMessage.setMessage(StringUtils.join(outputList, "\n"));
                // 分批获取进程的输出
                BufferedReader errorBufferedReader = new BufferedReader(new InputStreamReader(runProcess.getErrorStream()));

                List<String> errorOutpputList = new ArrayList<>();
                // 逐行读取
                String errorCompileOutputLine;
                while ((errorCompileOutputLine = errorBufferedReader.readLine()) != null) {
                    errorOutpputList.add(errorCompileOutputLine);
                }
                executeMessage.setErrorMessage(StringUtils.join(errorOutpputList, "\n"));
            }
            stopWatch.stop();
            executeMessage.setTime(stopWatch.getLastTaskTimeMillis());
        } catch (Exception e) {
            e.printStackTrace();
        }
        return executeMessage;
    }

    /**
     * 执行交互式进程获取信息
     *
     * @param runProcess 进程
     * @param opName     操作名称
     * @return
     */
    public static ExecuteMessage runInteractProcessAdnGetMessage(Process runProcess, String opName, String args) {
        ExecuteMessage executeMessage = new ExecuteMessage();
        try {
            // 向控制台输入程序
            OutputStream outputStream = runProcess.getOutputStream();

            OutputStreamWriter outputStreamWriter = new OutputStreamWriter(outputStream);
            String[] s = args.split(" ");
            outputStreamWriter.write(StrUtil.join("\n", s) + "\n");
            // 相当于按了回车，执行输入的发送
            outputStreamWriter.flush();

            // 分批获取进程的输出
            InputStream inputStream = runProcess.getInputStream();
            BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(inputStream));
            StringBuilder compileOutputStringBuilder = new StringBuilder();
            // 逐行读取
            String compileOutputLine;
            while ((compileOutputLine = bufferedReader.readLine()) != null) {
                compileOutputStringBuilder.append(compileOutputLine);
            }
            executeMessage.setMessage(compileOutputStringBuilder.toString());
            // 记得资源回收
            outputStreamWriter.close();
            inputStream.close();
            outputStream.close();
            runProcess.destroy();

        } catch (Exception e) {
            e.printStackTrace();
        }
        return executeMessage;
    }
}
```

