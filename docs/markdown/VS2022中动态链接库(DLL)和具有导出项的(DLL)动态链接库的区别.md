# VS2022中动态链接库(DLL)和具有导出项的(DLL)动态链接库的区别

![image-20220120124852882](https://cdn.jsdelivr.net/gh/zerobiubiu/Figure-bed/image-20220120124852882.png)

## 区别

1. 具有导出项的会生成dll和lib、还有头文件
2. 动态链库的话只有dll

## 文件位置

dll：项目目录\调试位数（X86默认无）\（Debug OR Release）\

lib：同上

头文件：项目目录\