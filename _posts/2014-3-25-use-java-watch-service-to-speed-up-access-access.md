---
layout : post
title : 应用 java nio监视来加速读取access
category : "java"
tags : [java,access,nio]
---

上一篇文章讲到使用java jdbc连接access数据库，然后实际使用中发现，因为没有连接池的问题，导致每次读取数据会消耗大量的事件来简历数据库连接，执行sql，然后关闭连接。

考虑到access数据库以文件的形式保存在文件系统中，可以使用java的nio提供的watchService来监视文件，当文件被修改时，读取数据保存到java缓存中。这样就能节省每次读取信息时消耗的大量垃圾事件。

逻辑如下：

1. 应用启动
2. 将access中的信息保存到AppConfig.java 的 List中.初始化List为NULL。
3. 修改List的get方法。判断List是否为NULL，非NULL直接返回，NULL则读取一次数据库，将返回的信息的赋值给List。同时开启新线程，监控数据库。
4. WatchDir 监控access数据库所在目录，当发生目标文件的修改事件时，执行读取数据库的代码，将返回值赋值给appConfig的list。

文件清单:

1. AppConfig.java 用来保存缓存信息。
2. WatchDir.java watchService的主要类。从ora官网java教程中的例子修改而来。
3. AccessDBUtil.java 读取数据库的类。

提速效果：从之前每次10s左右的读取消耗，提升到第一次读取`10s`，之后读取耗时`0.1s`的成果。


AppConfig.java
```java
package com.cyberdak.util.access;

import java.io.IOException;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;

import sy.model.access.OutBuy;
import sy.util.ConfigUtil;

/**
 * @author 亢伟楠
 *	时间:2015年3月24日下午7:11:49
 */
public class AppConfig {
		private static List<OutBuy> list = null;
		
		public static List<OutBuy> getList() {
			if(list == null){
				try {
					list = OutAccessDBUtil.getOutBuyList();
					new Thread(new Runnable() {
						@Override
						public void run() {
							System.out.println("缓存中list为null，开启watchService");
							Path dir = Paths.get(ConfigUtil.get("dataDir"));
							try {
								new WatchDir(dir, false).processEvents();
							} catch (IOException e) {
								e.printStackTrace();
							}
						}
					}).start();
					return list;
				} catch (SQLException e) {
					e.printStackTrace();
				}
			}
			return list;
		}

		public static void setList(List<OutBuy> list) {
			AppConfig.list = list;
		}
		
		

}
```

WatchDir.java
```java
package com.cyberdak.util.access;
/*
 * Copyright (c) 2008, 2010, Oracle and/or its affiliates. All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions
 * are met:
 *
 * - Redistributions of source code must retain the above copyright
 * notice, this list of conditions and the following disclaimer.
 *
 * - Redistributions in binary form must reproduce the above copyright
 * notice, this list of conditions and the following disclaimer in the
 * documentation and/or other materials provided with the distribution.
 *
 * - Neither the name of Oracle nor the names of its
 * contributors may be used to endorse or promote products derived
 * from this software without specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS
 * IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO,
 * THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR
 * PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT OWNER OR
 * CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL,
 * EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
 * PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR
 * PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF
 * LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING
 * NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
 * SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 */

import java.nio.file.*;

import static java.nio.file.StandardWatchEventKinds.*;
import static java.nio.file.LinkOption.*;

import java.nio.file.attribute.*;
import java.io.*;
import java.sql.SQLException;
import java.util.*;

import javax.servlet.jsp.jstl.core.Config;

import sy.util.ConfigUtil;

/**
 * Example to watch a directory (or tree) for changes to files.
 * 监视目录或者树内部文件改变的例子。
 */

public class WatchDir {

	private final WatchService watcher;
	private final Map<WatchKey,Path> keys;
	private final boolean recursive;
	private boolean trace = true;//是否开启追踪

	@SuppressWarnings("unchecked")
	static <T> WatchEvent<T> cast(WatchEvent<?> event) {
		return (WatchEvent<T>)event;
	}

	/**
	 * Register the given directory with the WatchService
	 * 在WatchService中注册目录
	 */
	private void register(Path dir) throws IOException {
//		WatchKey key = dir.register(watcher, ENTRY_CREATE, ENTRY_DELETE, ENTRY_MODIFY);
		WatchKey key = dir.register(watcher, ENTRY_MODIFY);//只监控修改事件
		if (trace) {
			Path prev = keys.get(key);
			if (prev == null) {
				System.out.format("register: %s\n", dir);
			} else {
				if (!dir.equals(prev)) {
					System.out.format("update: %s -> %s\n", prev, dir);
				}
			}
		}
		keys.put(key, dir);
	}

	/**
	 * Register the given directory, and all its sub-directories, with the
	 * WatchService.
	 * 在WatchService中注册目录以及它的子目录
	 */
	private void registerAll(final Path start) throws IOException {
		// register directory and sub-directories
		Files.walkFileTree(start, new SimpleFileVisitor<Path>() {
			@Override
			public FileVisitResult preVisitDirectory(Path dir, BasicFileAttributes attrs)
					throws IOException
			{
				register(dir);
				return FileVisitResult.CONTINUE;
			}
		});
	}

	/**
	 * Creates a WatchService and registers the given directory
	 * 创建WatchService，然后注册接收的目录
	 */
	WatchDir(Path dir, boolean recursive) throws IOException {
		this.watcher = FileSystems.getDefault().newWatchService();
		this.keys = new HashMap<WatchKey,Path>();
		this.recursive = recursive;

		if (recursive) {
			System.out.format("Scanning %s ...\n", dir);
			registerAll(dir);
			System.out.println("Done.");
		} else {
			register(dir);
		}

		// enable trace after initial registration
		this.trace = true;
	}

	/**
	 * Process all events for keys queued to the watcher
	 * 处理watcher的所有队列事件
	 */
	void processEvents() {
		for (;;) {

			// wait for key to be signalled
			WatchKey key;
			try {
				key = watcher.take();
			} catch (InterruptedException x) {
				return;
			}

			Path dir = keys.get(key);
			if (dir == null) {
				System.err.println("WatchKey not recognized!!");
				continue;
			}
			int count = 0;

			for (WatchEvent<?> event: key.pollEvents()) {
				WatchEvent.Kind kind = event.kind();

				// TBD - provide example of how OVERFLOW event is handled
				if (kind == OVERFLOW) {
					continue;
				}

				// Context for directory entry event is the file name of entry
				WatchEvent<Path> ev = cast(event);
				Path name = ev.context();
				Path child = dir.resolve(name);

				// print out event
				// 打印事件
				System.out.format("%s: %s,\n", event.kind().name(), child);
				if(event.kind().equals(ENTRY_MODIFY ) && child.toString().equals(ConfigUtil.get("outBuyAccess"))){
					try {
						System.out.println("期望数据库被修改，更新缓存.");
						AppConfig.setList(OutAccessDBUtil.getOutBuyList());
					} catch (SQLException e) {
						e.printStackTrace();
					}
				}

				// if directory is created, and watching recursively, then
				// register it and its sub-directories
				if (recursive && (kind == ENTRY_CREATE)) {
					try {
						if (Files.isDirectory(child, NOFOLLOW_LINKS)) {
							registerAll(child);
						}
					} catch (IOException x) {
						// ignore to keep sample readbale
					}
				}
			}

			// reset key and remove from set if directory no longer accessible
			boolean valid = key.reset();
			if (!valid) {
				keys.remove(key);

				// all directories are inaccessible
				if (keys.isEmpty()) {
					break;
				}
			}
		}
	}

	static void usage() {
		System.err.println("usage: java WatchDir [-r] dir");
		System.exit(-1);
	}

	public static void main(String[] args) throws IOException {
		// parse arguments
		// 解析参数
//		if (args.length == 0 || args.length > 2)
//			usage();
		boolean recursive = false;
//		int dirArg = 0;
//		if (args[0].equals("-r")) {
//			if (args.length < 2)
//				usage();
//			recursive = true;
//			dirArg++;
//		}

		// register directory and process its events
		Path dir = Paths.get(ConfigUtil.get("dataDir"));
		new WatchDir(dir, recursive).processEvents();
	}
}
```