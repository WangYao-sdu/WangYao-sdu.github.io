---
title: 光线追踪
tag: 计算机图形学
categories: 计算机图形学 openGL
---

/*
实验代码和实验报告在github的ray-tracing项目中，欢迎下载。
*/

## 一 ．实验要求
(1) 利用Phong shaing模型和Gouraud shading模型实现光线的追踪，画出光照模型。  
(2) 比较Phong shaing模型和Gouraud shading模型的异同，更好地理解光照两种模型。  
(3) 通过环境光、漫反射和镜面反射参数的设置，使场景更加真实自然。  
## 二． 完成情况
(1) 场景的绘制：由一个桌面和三个小球组成，三个小球分别是蓝色，红色和黑色。这里图形的绘制上个实验已经讲解的很清楚，不再累赘。  
(2) 定义点光源：光源所在的位置坐标和光源方向。  
(3) 编写程序，描绘光线照射小球时，人眼所看到的场景：物体的颜色，形状，光线直射物体时它所发生的镜面反射，
以及没有直射到的部分所受到的光的漫反射。  
(4) 小球在桌面的阴影的实现：递归地进行光线追踪，因此还需要设置追踪深度。不断的以新交点为起始点，以反射光方向为方向
进行在一个的求交点的过程中，直到到达我们设定的深度。
## 三．具体实验过程  
### 1. 配置opengl库函数  
### 2．学习具体需要使用的相关函数：
<pre><code>glPushMatrix();  
glPopMatrix();  
glScalef(0.5, 0.5, 0.5); //缩放函数  
glutSolidCube(1.0); //新建立方体  
glColor3f(1.0, 0.6, 0.4);//着色  
glTranslatef(0.0, 1.25, 0.0); //平移  
glRotatef((GLfloat)turn, 0.0, 1.0, 0.0);//旋转</code></pre>
### 3.构思需要的变量：  
（1）x,y,z坐标；  
（2）float yy = 0.0;//机器人在z轴方向行走的距离  
（3）int angle = 0;//机器人的方向，设置它可以左右旋转身体  
（4）int lshoulder = 0, lelbow = 0, rshoulder = 0, relbow = 0;//设置左右手臂与肩膀处的旋转角度和下手臂与上手臂的旋转角度  
（5）int lhips = 0, rhips = 0, lfoot = 0, rfoot = 0;//设置左右腿部与身体处的旋转角度和小腿与大腿处的旋转角度  
### 4.实现机器人的走动  
（1）实现机器人外形设计 
<pre><code>//头部设计：一个立方体  
void draw_head(void)  
{  
	glPushMatrix();  
	glTranslatef(0, 3.5, yy);  
	glTranslatef(0, 1, 0);  
	glutSolidCube(2);  
	glPopMatrix();  	
}  
//身体：一个立方体  
void draw_body(void)  
{  
	glPushMatrix();  
	glTranslatef(0, 1.5, yy);  
	glScalef(0.5, 1, 0.4);  
	glutSolidCube(4);  
	glPopMatrix();  
}  
//左手臂：两个立方体之间一个球体。右手臂相同，不再累赘  
void draw_leftshoulder(void)  
{  
	glPushMatrix();  
	//画一个立方体  
	glTranslatef(1.5, 3, yy);  
	glRotatef(lshoulder, 1, 0, 0);  
	glTranslatef(0, -0.5, 0);  
	glScalef(0.4, 1, 0.5);  
	glutSolidCube(2);  
	//画一个球体  
	glScalef(1 / 0.4, 1 / 1, 1 / 0.5);  
	glTranslatef(0, -1.4, 0);  
	glRotatef(lelbow, 1, 0, 0);  
	glutWireSphere(0.4, 200, 500);  
	//画一个立方体  
	glScalef(0.4, 1, 0.5);  
	glTranslatef(0, -1.4, 0);  
	glutSolidCube(2);  
	glPopMatrix();  
}  
//左腿：两个立方体之间一个球体。右腿相同，不再累赘  
void draw_leftfoot(void)  
{  
	glPushMatrix();  
	//画一个立方体  
	glTranslatef(-0.6, -0.6, yy);  
	glRotatef(lfoot, 1, 0, 0);  
	glTranslatef(0, -1, 0);  
	glScalef(0.4, 1, 0.5);  
	glutSolidCube(2);  
	//画一个球体  
	glScalef(1 / 0.4, 1 / 1, 1 / 0.5);  
	glTranslatef(0, -1.4, 0);  
	glRotatef(lhips, 1, 0, 0);  
	glutWireSphere(0.4, 200, 500);  
	glScalef(0.4, 1, 0.5);  
	//画一个立方体  
	glScalef(1, 1, 1);  
	glTranslatef(0, -1.4, 0);  
	glutSolidCube(2);  
	glPopMatrix();  
}</code></pre>
（2）通过按键，使机器人运动  
<pre><code>//按“0”使机器人整体左转  
	case '0':  
		angle = (angle + 5) % 360;  
		glutPostRedisplay();  
		break;  
	//按“1”使机器人整体左转  
	case '1':  
		angle = (angle - 5) % 360;  
		glutPostRedisplay();  
		break;  
	//按“2”使机器人协调一致行走  
	case '2':  
		//yy为z轴坐标，使机器人在-8到8之间运动，以使得它可以一直出现在屏幕中  
		if (yy < 8.0) {  
			yy += 0.05;  
		}  
		else {  
			yy = -8.0;  
		}  
		//左手臂旋转，达到设置的极值角度反向旋转，其他部位的旋转与此类似，不再累赘  
		if (flaglshoulder)  
		{  
			lshoulder += 5;  
			if (lshoulder >= 60)flaglshoulder = 0;  
		}    
		else    
		{   
			lshoulder -= 5;   
			if (lshoulder <= -60)flaglshoulder = 1;   
		}  
		glutPostRedisplay();</code></pre>
（3）具体操作：先长按“1”键使机器人左转90度，再长按“2”键使机器人行走。停止按键，机器人停止运动。  
## 四． 实验结果
(1)机器人外形图片：  
以下分别是机器人的正面图片和运动图片  
![Markdown](http://i4.buimg.com/591351/e27d3745a2e14555.png)  
![Markdown](http://i4.buimg.com/591351/710cc4d428cce60c.png)  
(2)实验效果视频如下：[优酷视频链接](http://v.youku.com/v_show/id_XMjY4MzQ2MzAzNg==)