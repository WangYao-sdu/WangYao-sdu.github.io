---
title: 行走的机器人
tag: 计算机图形学
categories: 计算机图形学 openGL
---

/*
实验代码和实验报告在github的walking-robot项目中，欢迎下载。
*/

## 一 ．实验要求

(1) 使用OPENGL库函数画一个由矩形立方体组成的机器人，在它的手臂和腿部的关节使用小球体代表。  
(2) 通过glTranslatef和glRotatef操作，实现机器人像真人一样行走：关节可动，行动过程中不散架。  
(3) 行走过程中手和腿的摆动角度通过参数限定显得自然，在人可运动的角度范围内。  
## 二． 完成情况

(1) 机器人的外形：头部、身体、手部和腿部用立方体表示，手部和腿部的关节处用圆球表示。  
要想使得机器人行走起来，使用opengl自带的平移函数，控制头、身体、左臂上关节、右臂上关节、左腿上关节、右腿上关节同步平移。  
(2) 机器人关节的变换用opengl自带的旋转函数glRotatef，设置好每个部位的旋转中心，比如：下手臂以上下手臂之间的球形关节为旋转
中心，整个手臂以上手臂与身体相连的肩膀处为旋转中心，腿部的大腿和小腿的旋转情况与手部类似。  
(3) 通过参数设置，实现了左右臂向相反方向摆动，并且上手臂的摆动范围为正负60度，下手臂只能上前摆动30度然后反向。大腿和左右臂
摆动同步，左右大腿分别对应右左手臂，以使机器人像真人一样行走不会顺拐。大腿的摆动范围为前后60度，而小腿与下手臂相反，只能
往后旋转30度之后反向旋转。这符合人的运动规律，显得更加自然。  
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
实验效果视频如下 [优酷视频链接](http://v.youku.com/v_show/id_XMjY4MzQ2MzAzNg==)