---
layout: post
title: "SVN 常用命令行操作"
date: 2016-01-19 10:07:10 +0800
comments: true
categories: 
---


# 一、SVN 常用命令行操作
虽然 git 已经很普及了，但 SVN 还是有很多团队在使用，本文不是 SVN 的详细教程，只是对 SVN 常用命令行操作的记录，以便在使用的时候不至于再去搜索用法。Mac 上搭建服务器和配置用户什么的参见[Mac下搭建svn服务器和XCode配置svn](http://blog.csdn.net/jjunjoe/article/details/8500996)。

***提示：***
<mark>美元符开头的都是变量，需要替换为实际的值。</mark>

## 1.0 本地 svn 服务的几个命令：
带点私货，本地 svn 服务的几个命令，没地方放，就放这了：
### 1.0.1 查看svn是否启动
	lsof -i :3690
	
### 1.0.2 查找所有svn启动的进程id
	ps aux | grep 'svn'
	
### 1.0.3 将pid替换为上面查到的进程id可以杀掉svn进程
	kill -9 pid

## 1.1 从 SVN 仓库导出项目
### 1.1.1 导出最新版本
	svn checkout $svn_server_path_to_project --username=$username --password=$password  $local_path_to_project

### 1.1.1 导出指定版本
	svn checkout -r$revison_number $svn_server_path_to_project --username=$username --password=$password  $local_path_to_project

## 1.2 将项目导入 SVN 仓库
	svn import $project_path_you_need_import $svn_server_path_for_project --username $username --password $password -m "$comment information"

## 1.3 查看 SVN 仓库内容
	svn ls $svn_server_path 

## 1.4 取消修改（还没有 commit 的情况）
### 1.4.1 取消文件修改
	svn revert $file_path
	
### 1.4.2 取消目录修改
	svn revert --depth=infinity $dir_path

## 1.5 提交修改
	svn commit $file_or_dir -m "$comment info"

## 1.6 回退到某版本

* 先更新到最新版本  
	
		svn update

* 查看日志,找到需要回退的版本  
	* 查看所有日志  
	
			svn log  
	
	* 查看指定数量日志  
		
			svn log -l $maximum_number_of_log

* 反向合并  

		svn merge -r rHEAD:$to_revision "$comment info"  

* 提交合并后的修改  

		svn ci $file_or_dir -m "$comment info" 

## 1.7 创建分支
	svn copy $trunk_path_of_project $branch_path_of_project -m "$comment info"

## 1.8 合并
## 1.8.1 从主干合并到分支
* cd 到本地分支目录：  

		cd $branch_path

* 合并主干代码：  
	* 合并指定的 `$reversion` 的修改：  
	
			svn merge -c $reversion1[,$reversion2,...] $trunk_path_of_project
	
	* 合并 `$from_revision` 到 `$to_revesion` 的修改：
			
			svn merge -r $from_reversion:$to_reversion $trunk_path_of_project

* 提交合并后的修改：  

		svn ci $file_or_dir -m "$comment info"

## 1.8.2 从分支合并到主干
跟从主干合并到分支差不多。  

* cd 到本地 trunk 目录：  

		cd $trunk_path

* 合并主干代码：  
	* 合并指定的 `$reversion` 的修改：  
	
			svn merge -c $reversion1[,$reversion2,...] $branch_path_of_project 
	
	* 合并 `$from_revision` 到 `$to_revesion` 的修改：
	
			svn merge -r $from_reversion:$to_reversion $branch_path_of_project

* 提交合并后的修改：  

		svn ci $file_or_dir -m "$comment info" 