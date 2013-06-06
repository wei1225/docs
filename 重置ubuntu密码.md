#重置Ubuntu密码

by can.    
2013/6/6

1. 开机，在GRUB处选择`recovery mode`
2. 在`Recovery Menu`处选择`Drop to root shell prompt`
3. 此时文件系统是只读的，输入`mount -rw -o remount /`重新挂载成读写
4. 这时可以用`passwd xxx`来修改xxx用户的密码了