# 创建第一个Qt程序

1. 点击创建项目后， 选择项目路径以及给项目起名称
2. 名称——不能有中文不能有空格
3. 路径——不能有中文路径
4. 默认创建有窗口类，myWidget, 基类有三种选择：QWidget、QMainWindow、QDlalog
5. main 函数。
   `QApplication a；`应用程序对象, 有且仅有一个
   `myWidget w；`实例化窗口对象
   `w.show()；`调用show函数显示窗口
   `return a.exec()；`让应用程序对象进入消息循环机制中，代码阻塞到当前行

# 控件

> 控件实现大都写在生成的widget.cpp内的
>
> `Widget::Widget(QWidget *parent):QWidget(parent)`
>
> 内

## 按钮控件

> 头文件　QPushButton

### 生成按钮

```C++
//创建一个按钮
QPushButton *btn=new QPushButton;
//按钮输出在this主窗口
btn->setParent(this);
//按钮文字
btn->setText("第一个");
//第二个按钮("按钮显示文字",按钮窗口)
QPushButton *btn2=new QPushButton("第二个",this);
```

### 按钮位置

```c++
//按钮位置(x,y)
btn2->resize(20,10);
```

### 按钮大小

```c++
//定义按钮的大小
btn2->move(50,20);
```



## 窗口控件

### 窗口大小

```c++
//窗口大小(宽,高)
resize(200,100);
//设置固定的窗口大小(宽,高)
setFixedSize(200,100);
```

### 窗口标题

```c++
//窗口标题
setWindowTitle("window");
```

# 信号和槽

## 事件处理

关闭窗口

```c++
//(信号发送者,发送的信号(函数的地址),接收信号者,处理信号的槽函数)
connect(btn2,&QPushButton::clicked,this,&QWidget::close);
```

# 使用总结

## QPushButton 使用总结

> pBtnTest	指	QPushButton *pBtnTest=new QPushButton;	中创建出来的

### 设置位置和大小

```c++
// 重新设定按钮的位置
pBtnTest->move(100, 50);

// 重新设定按钮的大小
pBtnTest->resize(80, 50);

// 设置按钮的位置和大小
pBtnTest->setGeometry(100, 50, 80, 50);
```

### 设置显示文本信息的字体

```c++
pBtnTest->setFont(QFont("宋体", 18));
```

### 根据文本长度自动调整大小

```c++
pBtnTest->setText("我是一个很长很长很长的文本");
    
// adjustSize()：自动调整控件的大小，以适应其内容；
pBtnTest->adjustSize();
```

### 设置按钮获取焦点

```c++
// 设置控件获取焦点
pBtnTest->setFocus();

// 获取控件是否具有焦点；如果控件有焦点，返回 true；
bool b = pBtnTest->hasFocus();
qDebug() << b;

// 清除控件的焦点
pBtnTest->clearFocus();
```

### 设置鼠标位于按钮区域时，光标的类型

```c++
pBtnTest->setCursor(QCursor(Qt::BusyCursor));
```

### 设置按钮的 禁用 和 启用

```c++
// 禁用控件
pBtnTest->setDisabled(true);

// 启用控件
pBtnTest->setEnabled(true);
```

### 设置按钮背景透明：即将按钮外观设为平铺

```c++
pBtnTest->setFlat(true);
```

### 设置在控件上按下 回车键 时，响应控件的 click 事件

```c++
pBtnTest->setDefault(true);
```

### 设置按钮上显示的图标

```c++
// 设置按钮上显示的图标
pBtnTest->setIcon(QIcon(":/Image/Luffy.png"));

// 设置图标的大小
pBtnTest->setIconSize(QSize(24, 24));
```

### 设置可选按钮

```c++
auto earMonitorSwitch = new QPushButton(this);
earMonitorSwitch->setCheckable(true);
earMonitorSwitch->setStyleSheet("QPushButton{border-image:url(./resource/audio/audio_setting/btn_earmonitor_close.png);}"
"QPushButton:checked{border-image:url(./resource/audio/audio_setting/btn_earmonitor_open.png);}");
```

### 设置不同状态下的通用CSS

```c++
pBtnTest->setStyleSheet("QPushButton{border-image:url(./resource/audio/audio_setting/close_normal.png);border:none;}"
		"QPushButton:hover{border-image:url(./resource/audio/audio_setting/close_hover.png);}"
		"QPushButton:pressed{border-image:url(./resource/audio/audio_setting/close_press.png);}");
```

### 设置按钮样式：前景色，背景色，边框等

```c++
// 定义初始样式集合
QStringList list;
list.append("color:white");                         // 前景色
list.append("background-color:rgb(85,170,255)");    // 背景色
list.append("border-style:outset");                 // 边框风格
list.append("border-width:5px");                    // 边框宽度
list.append("border-color:rgb(10,45,110)");         // 边框颜色
list.append("border-radius:20px");                  // 边框倒角
list.append("font:bold 30px");                      // 字体
list.append("padding:4px");                         // 内边距

// 设置按钮初始样式
pBtnTest->setStyleSheet(list.join(';'));                      
 
// 按钮按下时修改样式
list.replace(6, "font:bold 35px");
connect(pBtnTest, &QPushButton::pressed, [=](){
       pBtnTest->setStyleSheet(list.join(';'));
});

// 按钮弹起时恢复样式
list.replace(6, "font:bold 30px");
connect(pBtnTest, &QPushButton::released, [=](){
       pBtnTest->setStyleSheet(list.join(';'));
});
```

### 为按钮添加右键菜单

```c++
auto button = new QPushButton(u8"按钮");
button->setContextMenuPolicy(Qt::ActionsContextMenu);
if (type == AccomSoundType::mySound)
{
	auto deleteAction = new QAction(left);
	deleteAction->setText(u8"删除");
	connect(deleteAction, &QAction::triggered, this, [=]() {
		trace("删除");
	});
	button->addAction(deleteAction);
}
```

### 为按钮添加左键菜单

```c++
QPushButton* btn_room_setting = new QPushButton();
btn_room_setting->setObjectName("btn_room_setting");
auto settingMenu = new QMenu(this);
auto audioSettingAction = new QAction(u8"音频设置");
auto hostSettingAction = new QAction(u8"设置主持人");
auto adminSettingAction = new QAction(u8"设置管理员");
auto blackListAction = new QAction(u8"黑名单");
settingMenu->addAction(audioSettingAction);
settingMenu->addAction(hostSettingAction);
settingMenu->addAction(adminSettingAction);
settingMenu->addAction(blackListAction);
btn_room_setting->setMenu(settingMenu);
```

上面的方法会让按钮显示一个下拉的三角形图标，如果想取消这个图片可以用 qss 去除：

```c++
QPushButton::menu-indicator#btn_room_setting{
	image:none;
}
```



## 