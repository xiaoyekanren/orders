# 常用脚本

## 定时删除文件
<details>

``` shell
#/bin/bash
# 文件夹路径
backup_path='/opt/data/backups'
# 保留的文件数量
no_delete_quantity=30
# 检查间隔
check_interval=43200
```

<summary>点击查看代码</summary>


```
current_path=`pwd`
# 判断文件夹内文件的数量，大于保留文件的数量就删，小于就等待下次判断
check(){
	file_count_current=`ls -l $backup_path|grep '^-'|wc -l`
	used_space=`du -Ssh $backup_path|awk '{print $1}'`
	if (($file_count_current <= $no_delete_quantity));then
		echo "`date "+%Y-%m-%d %H:%M:%S"`,Now there are $file_count_current files,$used_space is used,no need to delete"
		continue
	else
		execute
	fi
}
# 删除
execute(){
	file_need_delete=`expr $file_count_current - $no_delete_quantity`
	# 核心，按照时间排序→xargs删除
	ls -l $backup_path|grep "^-"|sort -k 6,7|head -n $file_need_delete|awk '{print $9}'|xargs rm -rf
	echo "`date "+%Y-%m-%d %H:%M:%S"`,Now there are $file_count_current files,Already delete $file_need_delete files"
}
# 输出pid
echo "pid=$$"
# 主循环，睡眠→进目录→判断→是否执行→回之前目录→睡眠
while true;do
	sleep $check_interval
	cd $backup_path	
	check
	cd $current_path
done
```

</details>

