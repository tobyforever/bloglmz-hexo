---
title: GPT教你写代码：如何删除java源代码的空行和注释
date: 2023-04-13 20:46:51
tags: [java]
toc: true
published: true
---

申请软著有个奇葩需求，要求源代码不能带有空行和注释。

用GPT写了一个小程序来处理：

```java
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;
import java.nio.file.*;
import java.nio.file.attribute.BasicFileAttributes;
import java.util.regex.Pattern;

public class RemoveCommentsAndEmptyLines {
    private static final Pattern SINGLE_LINE_COMMENT_PATTERN = Pattern.compile("^\\s*//.*$");
    private static final Pattern MULTI_LINE_COMMENT_START_PATTERN = Pattern.compile(".*?/\\*.*");
    private static final Pattern MULTI_LINE_COMMENT_END_PATTERN = Pattern.compile(".*?\\*/.*");

    public static void main(String[] args) {
        //要处理的java源代码目录，会递归处理里面的java文件
        String srcpath="E:\\workspaceLMZ\\jtlsh\\flink-homwork\\src\\main\\java\\com\\linmingzhe";

        Path sourceDirectory = Paths.get(srcpath);
        processSourceFilesRecursively(sourceDirectory);
    }

    public static void processSourceFilesRecursively(Path sourceDirectory) {
        try {
            Files.walkFileTree(sourceDirectory, new SimpleFileVisitor<Path>() {
                @Override
                public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
                    if (file.toString().endsWith(".java")) {
                        removeCommentsAndEmptyLines(file);
                    }
                    return FileVisitResult.CONTINUE;
                }
            });
        } catch (IOException e) {
            System.err.println("Error walking the file tree: " + e.getMessage());
        }
    }

    public static void removeCommentsAndEmptyLines(Path filePath) {
        Path tempFilePath = Paths.get(filePath.toString() + ".tmp");
        try (BufferedReader reader = new BufferedReader(new FileReader(filePath.toFile()));
             BufferedWriter writer = new BufferedWriter(new FileWriter(tempFilePath.toFile()))) {

            boolean inMultiLineComment = false;
            String line;
            while ((line = reader.readLine()) != null) {
                if (!inMultiLineComment && !isSingleLineComment(line)) {
                    if (isMultiLineCommentStart(line)) {
                        inMultiLineComment = true;
                    }
                    line = removeInlineComment(line);

                    if (!line.trim().isEmpty()) {
                        writer.write(line);
                        writer.newLine();
                    }
                } else if (inMultiLineComment && isMultiLineCommentEnd(line)) {
                    inMultiLineComment = false;
                    line = line.substring(line.indexOf("*/") + 2);

                    line = removeInlineComment(line);
                    if (!line.trim().isEmpty()) {
                        writer.write(line);
                        writer.newLine();
                    }
                }
            }
        } catch (IOException e) {
            System.err.println("Error processing the file: " + e.getMessage());
            return;
        }

        // Replace the original file with the filtered content
        try {
            Files.delete(filePath);
            Files.move(tempFilePath, filePath);
        } catch (IOException e) {
            System.err.println("Error replacing the original file: " + e.getMessage());
        }
    }


    private static boolean isSingleLineComment(String line) {
        return SINGLE_LINE_COMMENT_PATTERN.matcher(line).matches();
    }

    private static boolean isMultiLineCommentStart(String line) {
        return MULTI_LINE_COMMENT_START_PATTERN.matcher(line).matches();
    }

    private static boolean isMultiLineCommentEnd(String line) {
        return MULTI_LINE_COMMENT_END_PATTERN.matcher(line).matches();
    }

    private static String removeInlineComment(String line) {
        int singleLineCommentStart = line.indexOf("//");
        if (singleLineCommentStart != -1) {
            line = line.substring(0, singleLineCommentStart);
        }

        int multiLineCommentStart = line.indexOf("/*");
        if (multiLineCommentStart != -1) {
            int multiLineCommentEnd = line.indexOf("*/", multiLineCommentStart);
            if (multiLineCommentEnd != -1) {
                return line.substring(0, multiLineCommentStart) + line.substring(multiLineCommentEnd + 2);
            } else {
                return line.substring(0, multiLineCommentStart);
            }
        }
        return line;
    }
}

```


中间有些小bug，提醒GPT改了3、4次，GPT都能按要求修改正确，的确很聪明。