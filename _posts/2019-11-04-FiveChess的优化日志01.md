---
layout:     post
title:      FiveChess的优化
subtitle:   优化FiveChess的一些心得记录，包含框架布局、算法改进和代码优化。
date:       2019-11-04
author:     LJJ
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - java
---

### 小记demo

### 第一天
今天开始的对FiveChess这个小demo进行修正，顺便巩固一下java基础知识，练习和回顾一些Swing和awt的控件布局。

#### 修正棋子图像边缘锯齿化

现象展示:

![修改前.PNG](https://i.loli.net/2019/11/04/YrthpXosDePuJnH.png)

去除边缘锯齿化的效果：

![修改后.PNG](https://i.loli.net/2019/11/04/hYkyrdvOxHiWRpC.png)

- 图像锯齿化的原因

绘棋子代码：

    public void paintChess(Graphics g){
    		for(int i=0;i<chessArray.length;i++){
    			for(int j=0;j<chessArray[i].length;j++){
    				if(chessArray[i][j] == 1){
    					g.setColor(Color.WHITE);
    					g.fillOval(i * SIZE + X - CHESS / 2, j * SIZE + Y - CHESS / 2, CHESS, CHESS);
    				}else if(chessArray[i][j] == -1){
    					g.setColor(Color.BLACK);
    					g.fillOval(i * SIZE + X - CHESS / 2, j * SIZE + Y - CHESS / 2, CHESS, CHESS);
    				}
    			}
    		}
    	}
    }

锯齿边(Jagged Edge)是由顶点数据像素化之后成为片段而导致的。详细的说明参考[抗锯齿](https://learnopengl-cn.readthedocs.io/zh/latest/04%20Advanced%20OpenGL/11%20Anti%20Aliasing/)

修改的办法很简单，Java提供了更高级的Graphics2D库来进行绘图，这个库里边就有可以消除掉图像边缘锯齿化的实现方法。看下面这段代码：

    //获取画布，即是棋盘开始画
    Graphics g = gp.getGraphics();
    Graphics2D g2d = (Graphics2D) g;
    		g2d.setRenderingHint(RenderingHints.KEY_ANTIALIASING,RenderingHints.VALUE_ANTIALIAS_ON);
    //public abstract class Graphics2D extends Graphics

只需要在获取棋盘之后，开始画棋子之前将Graphics类型强制转成Graphics2D类型，再调用setRenderingHint()方法即可。其中抽象类Graphics2D继承自Graphics类，所以用起来非常方便。  
这里主要是讲开发，就不过多地扩展到图像处理方面的知识点了。

- 修复赢得棋局后仍可下棋的bug

现象展示：

![下棋修改前.PNG](https://i.loli.net/2019/11/04/j15vKhCDYGyTBaJ.png)

可以观察到这个时候黑棋已经获胜了，弹出了获胜窗口。

![下棋修改前1.PNG](https://i.loli.net/2019/11/04/8uci4gJElY3CDIh.png)

此时白棋还可落子，然后又现实白子赢了。  
这是由于弹出子窗口没有锁定。修改后的代码如下：

    public void winFrame(int winFlag) {
    		JDialog frame = new JDialog();//构造一个新的JFrame，作为新窗口。
            frame.setBounds(// 设置对话边框
                    new Rectangle(400,350,500,200)
                );
            JLabel jl = new JLabel(); // 注意类名别写错了。
            frame.getContentPane().add(jl);
            
            String text = ""; //初始化
            
            if(winFlag==1) {
            	//设置对话窗口文字内容
                 text = "<html>" + 
                		"<body>" + 
                		"<p align=\"center\">" + 
                		"白棋获胜" + 
                		"<br>" + 
                		"关闭对话即可继续操作！" + 
                		"</p>" + 
                		"</body>" + 
                		"</html>";
            }else {
            	text = "<html>" + 
                		"<body>" + 
                		"<p align=\"center\">" + 
                		"黑棋获胜" + 
                		"<br>" + 
                		"关闭对话即可继续操作！" + 
                		"</p>" + 
                		"</body>" + 
                		"</html>";
    		}
            
            jl.setText(text);
            //设置字体样式：字体类型，字体加粗，字体大小
            jl.setFont(new java.awt.Font("Dialog", 1, 20));
            jl.setVerticalAlignment(JLabel.CENTER);
            jl.setHorizontalAlignment(JLabel.CENTER);// 注意方法名别写错了。
            //设置模式类型。
            frame.setModalityType(Dialog.ModalityType.APPLICATION_MODAL);
            //参数 APPLICATION_MODAL：阻塞同一 Java 应用程序中的所有顶层窗口（它自己的子层次
            frame.setVisible(true);
    	}

这里遇到的问题也算比较多，最为麻烦的应该就是弹出窗口的那两行字的设计了。原先的排版我只是按照下面这样设置：

     jl.setText(“白棋获胜，关闭对话即可继续操作”);
    
但是很显然，弹出来不好看，于是我决定将这段文字换行居中。由于弹出窗口的文字是按照Html格式排版的，就改为了上述代码形式，用段落居中标签解决了文字排列不好看的问题。  
实现效果图：

![子弹窗修改后.PNG](https://i.loli.net/2019/11/04/G2dYlrmnwiHBNy4.png)

此时弹出子弹窗后是点击不了棋盘界面的，只有将对话框关闭才可以进行后续操作。

- 解决有边框按钮赘余，部分代码冗余

现象展示：

![边框.PNG](https://i.loli.net/2019/11/04/pBzRWSdQsXaIYZx.png)

明显可以看到有一个开始游戏的按钮，操作过程如下：
>  点击人人对战/人机对战，再点击开始游戏，再落子下棋。

我个人觉得这个流程可以被简化，仔细想一下，开始游戏这个按钮根本没有存在的意义。那么我们要做的就是在界面布局上将这个按钮给删掉。  
先看到实现布局的原始代码;


		GamePanel gp = new GamePanel(chessArray);
		//设置棋盘背景色
		gp.setBackground(new Color(139,139,122));
		jf.add(gp,BorderLayout.CENTER);
		//设置东边的属性
		JPanel jp=new JPanel();
		jp.setBackground(new Color(245,245,245));
		jf.add(jp,BorderLayout.EAST);
		//设置东边流式布局的属性
		java.awt.FlowLayout fl=new java.awt.FlowLayout(5,5,60);
		jp.setLayout(fl);
		Dimension di=new Dimension(120,0);
		jp.setPreferredSize(di);
		//设置东边的按钮及按钮的属性
		javax.swing.JButton jbu1=new javax.swing.JButton("开始游戏");
		Dimension di1=new Dimension(100,60);
		jbu1.setPreferredSize(di1);
		jp.add(jbu1);
		javax.swing.JButton jbu2=new javax.swing.JButton("人人对战");
		Dimension di2=new Dimension(100,60);
		jbu2.setPreferredSize(di2);
		jp.add(jbu2);
		javax.swing.JButton jbu3=new javax.swing.JButton("人机对战");
		Dimension di3=new Dimension(100,60);
		jbu3.setPreferredSize(di3);
		jp.add(jbu3);
		javax.swing.JButton jbu4=new javax.swing.JButton("悔棋");
		Dimension di4=new Dimension(100,60);
		jbu4.setPreferredSize(di4);
		jp.add(jbu4);
		javax.swing.JButton jbu5=new javax.swing.JButton("重新开始");
		Dimension di5=new Dimension(100,60);
		jbu5.setPreferredSize(di5);
		jp.add(jbu5);
        
        	//设置按钮的动作监听器
		jbu1.addActionListener(mouse);
		jbu2.addActionListener(mouse);
		jbu3.addActionListener(mouse);
		jbu4.addActionListener(mouse);
		jbu5.addActionListener(mouse);

可以看到，很多对象的建立是没有必要的，而且写法也比较使代码赘余。于是将代码加以精简，用Arraylist来储存生成的四个对象，之后再进行强制转型，如下：

第一部分：

    //设置选项边框流式布局的属性
    		jp.setLayout(new java.awt.FlowLayout(4,4,70));
    		jp.setPreferredSize(new Dimension(120,0));
    		 
    		//设置按钮及其属性
    		//Object gameButton[];
    		ArrayList<Object> gameButton = new ArrayList<Object>();
    		String strButton[] = new String[4];
    		strButton[0] = "人人对战";
    		strButton[1] = "人机对战";
    		strButton[2] = "悔棋";
    		strButton[3] = "重新开始";
    		
    		for(int i=0;i<4;i++) {
    			gameButton.add(new javax.swing.JButton(strButton[i]));
    			((JComponent) gameButton.get(i)).setPreferredSize(new Dimension(100,60));
    			jp.add((JComponent) gameButton.get(i));
    		}
    		
        		//设置时间监听器
    		for(int i=0;i<4;i++) {
    			((JButton) gameButton.get(i)).addActionListener(mouse);
    		}		

第二部分原先代码：

    // 白棋获胜后的界面
    	public void bwin() {
    		javax.swing.JFrame jf = new javax.swing.JFrame();
    		java.awt.FlowLayout fl = new java.awt.FlowLayout();
    		jf.setLayout(fl);
    		javax.swing.JLabel jlb = new javax.swing.JLabel("白棋获胜");
    		jlb.setFont(new java.awt.Font("Dialog", 1, 100));
    		// “dialog”代表字体，1代表样式(1是粗体，0是平常的）15是字号
    		jf.setSize(500, 200);
    		jf.add(jlb);
    		// javax.swing.JButton jbu=new javax.swing.JButton("在来一局");
    		// Dimension di=new Dimension(100,60);
    		// jbu.setPreferredSize(di);
    		// jf.add(jbu);
    		jf.setLocationRelativeTo(null);
    		jf.setDefaultCloseOperation(3);
    		jf.setVisible(true);
    	}
    
    	// 黑棋获胜后的界面
    	public void hwin() {
    		javax.swing.JFrame jf = new javax.swing.JFrame();
    		java.awt.FlowLayout fl = new java.awt.FlowLayout();
    		jf.setLayout(fl);
    		javax.swing.JLabel jlb = new javax.swing.JLabel("黑棋获胜");
    		jlb.setFont(new java.awt.Font("Dialog", 1, 100));
    		// “dialog”代表字体，1代表样式(1是粗体，0是平常的）15是字号
    		jf.setSize(500, 200);
    		jf.add(jlb);
    		// javax.swing.JButton jbu=new javax.swing.JButton("在来一局");
    		// Dimension di=new Dimension(100,60);
    		// jbu.setPreferredSize(di);
    		// jf.add(jbu);
    		jf.setLocationRelativeTo(null);
    		jf.setDefaultCloseOperation(3);
    		jf.setVisible(true);
    	}

修改后的代码已经在上面了，直接将两个函数合在了一起，并且解决了弹窗问题。  
经过修改后，源先的代码看起来就很清晰了。所以注意到：  
数据结构和算法的使用是会对程序的性能进行提升的，平时在写程序的时候需要考虑这一点。  
修改后的布局：

![边框修改后.PNG](https://i.loli.net/2019/11/04/FzEN2eCDu1gx3r8.png)

- 一些遗忘Eclipse的操作

快捷键 Shift+Alt+R键批量更换变量名  
Shift+Ctrl+/ 多行注释  
Shift+Ctrl+\ 取消多行注释
