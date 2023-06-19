---
layout: post
title: File Calculator
date: 2022-12-12 12:00:00
categories: [java]
---
 
# Project Overview

I decided I wanted to create a non-conventional calculator. So, I created one that takes up no space (taking up zero bytes), but how does one create a zero-byte calculator?

The code shown here has been modified to make it easier to explain. If you would like to check it out, the full code is available on GitHub: [Calculator](https://github.com/aidankeighron/Calculator){:target="\_blank"}

<details>
<summary><b>Table of Contents</b></summary>
<ul>
<li><a href="#project-overview">Project Overview</a></li>
<li><a href="#file-system">File System</a></li>
<li><a href="#generation">Generation</a></li>
<li><a href="#deletion">Deletion</a></li>
<li><a href="#final-thoughts">Final Thoughts</a></li>
<li><a href="#on-complexity">O(n) Complexity</a></li>
</ul>
</details>
 
## File System

Why can't we just do this?:

```java
public int add(int x1, int x2) {
    return x1 + x2;
}
```

Because it takes up space. For this project, we are trying to limit the space of whatever does the calculations i.e., `return x1 + x2`. Nearly every programming language except for a few esoteric languages has a file size that is greater than zero bytes. So how is creating a zero-byte calculator possible? First, let me clarify a few things. For this project, there will be a program (written in java for no particular reason) to generate this calculator and perform operations with it. The actual calculations will still be done with zero bytes but we will be using a program to interact with it.
 
Folders and empty files "technically" take up zero space (They do take up space in your filesystem's metadata files but we are talking about their size according to their properties). Because of that, we will be using both folders and empty files to create the calculator. As much as I would love to make a graphing calculator out of folders and files, I don't want my computer to spend the rest of its life generating them. So let's add some restrictions so we are not generating millions of folders and files.
 
- Calculable numbers are from 0 and 1000
- The only operation available is addition

 
Here is a look at the file system:

```json
Calculator/
├── 0
├── 1
├── 2
│   ├── 0
│   └── 1
│       └── 3 (file)
├── 3
```
 
Using a structure like this we store the result of the previous two directories as the name of a file. To find the result of a calculation e.g., to find the solution to 2 + 1 you find the name of the file in the /2/1/ directory (the file would be /2/1/3). The first dimension of folders is the rows and the second dimension of folders is the columns:
 
```java
// Get answer
File numberFile = new File(firstNumber + "/" + secondNumber);
System.out.println("Your answer is: " + numberFile.list()[0]);
```

## Generation
 
But before we get the answer we need to generate the files. To make it faster we will [multithread](https://www.geeksforgeeks.org/multithreading-in-java/){:target="\_blank"}. Without multithreading, it took me 30 minutes to generate all the files and folders so I think it is necessary. Now, let's take a look at the `CalcThread` class which is the thread that will create parts of the matrix:
```java
public static class CalcThread extends Thread {
   
    private int startRow;
    private int endRow;
    private int numColumns;
   
    public CalcThread(int startRow, int endRow, int numColumns) {
        this.startRow = startRow;
        this.endRow = endRow;
        this.numColumns = numColumns;
    }
   
    public void run() {
        try {
            for (int i = startRow; i < endRow; i++) {
                new File("Numbers" + "\\" + i).mkdir();
                for (int j = 0; j < numColumns; j++) {
                    new File("Numbers" + "\\" + i + "\\" + j).mkdir();
                    new File("Numbers" + "\\" + i + "\\" + j + "\\" + (i + j)).createNewFile();
                }
            }
        } catch (Exception e) {e.printStackTrace();}
    }
}
```
This thread's goal is to generate a 2d matrix of folders with rows from `startRow` to `endRow` and a column length of `numColumns`. A file is created at each index with a name that is the sum of the column and row. For example, if the row is 3 and the column is 2 the filename would be 5.
 
Next, let's take a look at the `calculateNumbers()` method which will allocate rows to each thread.
 
``` java
private static void calculateNumbers(int numThreads) throws Exception {        
    int rowPerThread = maxNum/numThreads;
    CalcThread[] threads = new CalcThread[numThreads];
    for (int i = 0; i < numThreads-1; i++) {
        threads[i] = new CalcThread((rowPerThread * i), (rowPerThread * (i + 1)), maxNum);
        threads[i].start();
    }
    threads[threads.length-1] = new CalcThread(maxNum - rowPerThread, maxNum, maxNum);
    threads[threads.length-1].start();
    for (int i = 0; i < numThreads; i++) {
        threads[i].join();
    }
}
```
* `maxNum` is the total range of calculable numbers (e.g., You can add together numbers from `0` to `maxNum-1`)
 
Here we figure out how many rows to give each thread. We are writing a special case for the last thread so we don't miss any numbers due to rounding. If we run this program for numbers between 0 and 1000, it takes 1,003,002 folders, 1,002,001 files, a boot-up time of 6.5 minutes, and a shutdown time of 7.5 minutes to generate everything using 250 threads. Going from 30 minutes to 6.5 minutes shows you just how powerful multithreading is, the fact that we are using 250 of them also helps. We did overshoot our goal of not generating millions of files but we got pretty close.
 
## Deletion
 
Optionally you can delete everything when you are done by just right-clicking on the folder and pressing delete. But, let's do it with code which is faster when we multithread.
 
```java
public static void deleteNumbers(int numThreads) throws Exception {
    int rowPerThread = maxNum/numThreads;
    DeleteThread[] threads = new DeleteThread[numThreads];
    for (int i = 0; i < numThreads-1; i++) {
        threads[i] = new DeleteThread((rowPerThread * i), (rowPerThread * (i + 1)), maxNum);
        threads[i].start();
    }
    threads[threads.length-1] = new DeleteThread(maxNum - rowPerThread, maxNum, maxNum);
    threads[threads.length-1].start();
    for (int i = 0; i < numThreads; i++) {
        threads[i].join();
    }
    numberFolder.delete();
}
 
public static class DeleteThread extends Thread {
   
    private int startRow;
    private int endRow;
    private int numColumns;
   
    public DeleteThread(int startRow, int endRow, int numColumns) {
        this.startRow = startRow;
        this.endRow = endRow;
        this.numColumns = numColumns;
    }
   
    public void run() {
        try {
            for (int i = startRow; i < endRow; i++) {
                for (int j = 0; j < numColumns; j++) {
                    new File("Numbers\\" + i + "\\" + j + "\\" + (i+j)).delete();
                    new File("Numbers\\" + i + "\\" + j).delete();
                }
                new File("Numbers\\" + i).delete();
            }
        } catch (Exception e) {e.printStackTrace();}
    }
}
```
 
When deleting files with code, we need to delete everything recursively because java only allows us to delete empty folders. Essentially we are doing the same thing we did when creating the files and folders but in reverse.
 
## Final Thoughts
 
This project was not meant as a serious attempt at creating an efficient calculator. In reality, this calculator takes up more space than a conventional calculator and takes longer to find the answer. [Analyzing](https://www.geeksforgeeks.org/analysis-algorithms-big-o-analysis/){:target="\_blank"} it, we find it has a size complexity of $$O(2n^2 + n)$$ (simplifying down to $$O(n^2)$$) and I don't even want to spend the time calculating time complexity. Altogether it was a fun project and I enjoyed creating it and I hope you enjoyed reading about it.
 
## $$O(n)$$ complexity
 
```java
for (int i = 0; i < size; i++) {
    new File("Numbers" + "\\" + i).mkdir(); // 1st Dimension of folders (size)
    for (int j = 0; j < numColumns; j++) {
        new File("Numbers" + "\\" + i + "\\" + j).mkdir(); // 2nd Dimension of folders (size * numColumns)
        new File("Numbers" + "\\" + i + "\\" + j + "\\" + (i + j)).createNewFile(); // Files (size * numColumns)
    }
}
```
I will talk a bit about how I found the $$O(n)$$ size complexity for this project. N is the size of calculable numbers i.e., calculable numbers from 0 to 1000 would have an N of 1001. The $$O(n)$$ for the files is simply just `size * numColumns` as seen above and the same is true for the 2nd dimension of files with the 1st dimension being just `size`.
 
Full code: [Calculator](https://github.com/aidankeighron/Calculator){:target="\_blank"}