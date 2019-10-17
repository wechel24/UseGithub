# OpenGL ES渲染obj 3D模型总结

### GLSurfaceView类:

​			用于呈现3D模型，通过GLRender对象绘制3D模型，在Activity中进行界面呈现。

​			完成的功能包括：模型文件的加载、手指触摸事件（旋转、缩放）。

​			

```java
旋转：
	// 监听MotionEvent事件触发
	ACTION_DOWN: 检测当前手指触摸点位置
	ACTION_MOVE: 检测当前移动后手指停留点位置，与之前的点相减就是 X、Y 方向上的位移量
	             检测X 、Y分量上的位移，（X > Y）? 向X轴位移相应增量 : 向Y轴位移相应增量
	ACTION_UP:   单指离开，因为在双指缩放后，也是单指离开，所以在此需要更新缩放比
				setScale{
					scale_now = scale_rember;
					scale_rember = 1.0f;
					scale = 1.0f;
				}
	             
缩放：
	ACTION_POINTER_DOWN:检测两个手指间的距离  
						pinchStartDistance = getPinchDistance(envent)
    ACTION_MOVE：检测当前两个手指间的距离
    					缩放比：scale = getPinchDistance(envent) / pinchStartDistance
    ！！！ 注意，双指缩放后，肯定是单指离开，所以在旋转的ACTION_UP中需要更新缩放比;
		  外部控制缩放（我们的语音控制）：需要先setScale更新缩放相关参数后，再改变缩放比scale；
```



### GLRender类:

​			实现模型的实际绘制，主要包含三个重载方法：

​			

```java
onSurfaceCreated(GL10 gl,EGLConfig config)
{
    /*
    	模型被创建时候，回调该方法
    	设置光源（环境光、背景光）
    */ 
}
```

```java
onSurfaceChanged(GL10 gl,int width,int height)
{
	/*
		SurfaceView组件大小发生改变时，回调该方法
		设置3D视图窗口的大小及位置
	*/
}
```

```java
onDrawFrame(GL10 gl)
{
    /*
    	前两个方法都是在初始化时调用，onDrawFram会一直不断的被加载，如果外部改变参数，则模型也会相应的发生改变；
    	绘制3D模型：模型翻转（x轴、y轴、1—3对角线）
        			gl.glRotatef(Degreen,x:1,y:0,z:0)   绕x轴旋转Degreen角度;
        			可进行翻转来改变模型的面向;
        		  模型大小尺寸
        		    gl.glScalef(scale_rember，scale_rember,scale_rember)
        		    通过设置scaleMax,scaleMin固定值，来限制scale_rember在限制范围内，可以限制模型					 的缩放域。
        		  绘制模型纹理
        		    需要使用GL10对象。
    */
}
```

### 程序逻辑：

3D模型渲染部分：

读取模型文件是时间 >> 模型绘制的时间，因此将绘制与模型加载独立开。

利用AsyncTask,一个轻量级异步类（抽象类），重写 onPreExecute()，doInBackground(Object... objects)，onPostExecute(Graphics graphics)三个方法，分别为：任务处理前，执行耗时任务，任务处理结束。

耗时任务：

​			obj数据加载、mtl文件加载，封装成一个Graphics对象；

Graphics对象包括：Mtl列表和GraphicsNode列表，就是模型坐标和纹理贴图信息；

当模型文件加载结束之后，GLrender利用该Graphics对象的draw方法，绘制模型到SurfaceView进行UI显示。





​		