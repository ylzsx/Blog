- 问题一：删除桌面文件夹时，提示找不到文件位置，删除不掉

  解决方式：写一个如下批处理文件，将删除不掉的文件夹拖到该批处理文件上

  ```bat
  del /f /a /q \\?\%1
  rd /s /q \\?\%1
  ```