# WaterMaskDemo
Android 给图片加上水印（支持logo+文字）吧！

现在我们想要往图片上打上水印，该水印应符合这样的需求的：
1. 支持logo+文字；
2. 文字信息支持多行展示；
3. 用户可以选择水印在图片上的生成位置（左上、右上、右下和左下）。

粗略的结构图低配版大概就长这样...

![水印结构图.png](http://upload-images.jianshu.io/upload_images/817079-a023ab2dfc726bb8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

现在提供这样的一种思路去实现这一个需求，我们可以通过自定义一个view，view的布局中包含logo、公司名称和相关信息，这个view就是我们要打上图片的水印。

这样的一个view其实是一个自定义组合布局，关于如何实现组合布局的自定义view，可以参考这篇文章：[Android 自定义View实践之组合控件实现布局的复用](http://www.jianshu.com/p/d378523a00d9)

有了水印的view之后，我们就可以利用以下这个方法，得到水印的view的Bitmap。

    /**
     * 将一个view转换为Bitmap
     * @param view
     * @return
     */
    public static Bitmap convertViewToBitmap(View view){
        view.measure(View.MeasureSpec.makeMeasureSpec(0, View.MeasureSpec.UNSPECIFIED), View.MeasureSpec.makeMeasureSpec(0, View.MeasureSpec.UNSPECIFIED));
        view.layout(0, 0, view.getMeasuredWidth(), view.getMeasuredHeight());
        view.buildDrawingCache();
        Bitmap bitmap = view.getDrawingCache();
        return bitmap;
    }


得到水印的view的Bitmap之后，处理一下尺寸，保持水印的尺寸在合理范围内。

    //根据原图处理要生成的水印的宽高
        float width = sourBitmap.getWidth();
        float height = sourBitmap.getHeight();
        float be = width / height;

        if ((float) 16 / 9 >= be && be >= (float) 4 / 3) {
            //在图片比例区间内16;9~4：3内，将生成的水印bitmap设置为原图宽高各自的1/5
            waterBitmap = WaterMaskUtil.zoomBitmap(waterBitmap, (int) width / 5, (int) height / 5);
        } else if (be > (float) 16 / 9) {
            //生成4：3的水印
            waterBitmap = WaterMaskUtil.zoomBitmap(waterBitmap, (int) width / 5, (int) width*3 / 20);
        } else if (be < (float) 4 / 3) {
            //生成4：3的水印
            waterBitmap = WaterMaskUtil.zoomBitmap(waterBitmap, (int) height*4 / 15, (int) height / 5);
        }

然后将它按照要求绘制在原图上，提供生成左上、右上、右下和左下四个位置的水印各自的方法，关键代码如下：

    /**
     * 设置水印图片在左上角
     * @param src
     * @param watermark
     * @param paddingLeft
     * @param paddingTop
     * @return
     */
    public static Bitmap createWaterMaskLeftTop(
            Context context, Bitmap src, Bitmap watermark,
            int paddingLeft, int paddingTop) {
        return createWaterMaskBitmap(src, watermark,
                dp2px(context, paddingLeft), dp2px(context, paddingTop));
    }

    /**
     * 设置水印图片在右下角
     * @param src
     * @param watermark
     * @param paddingRight
     * @param paddingBottom
     * @return
     */
    public static Bitmap createWaterMaskRightBottom(
            Context context, Bitmap src, Bitmap watermark,
            int paddingRight, int paddingBottom) {
        return createWaterMaskBitmap(src, watermark,
                src.getWidth() - watermark.getWidth() - dp2px(context, paddingRight),
                src.getHeight() - watermark.getHeight() - dp2px(context, paddingBottom));
    }

    /**
     * 设置水印图片到右上角
     * @param src
     * @param watermark
     * @param paddingRight
     * @param paddingTop
     * @return
     */
    public static Bitmap createWaterMaskRightTop(
            Context context, Bitmap src, Bitmap watermark,
            int paddingRight, int paddingTop) {
        return createWaterMaskBitmap( src, watermark,
                src.getWidth() - watermark.getWidth() - dp2px(context, paddingRight),
                dp2px(context, paddingTop));
    }

    /**
     * 设置水印图片到左下角
     * @param src
     * @param watermark
     * @param paddingLeft
     * @param paddingBottom
     * @return
     */
    public static Bitmap createWaterMaskLeftBottom(
            Context context, Bitmap src, Bitmap watermark,
            int paddingLeft, int paddingBottom) {
        return createWaterMaskBitmap(src, watermark, dp2px(context, paddingLeft),
                src.getHeight() - watermark.getHeight() - dp2px(context, paddingBottom));
    }

    /**
     * 绘制水印图片
     * @param src 原图
     * @param watermark 水印
     * @param paddingLeft
     * @param paddingTop
     * @return
     */
    private static Bitmap createWaterMaskBitmap(Bitmap src, Bitmap watermark,
                                                int paddingLeft, int paddingTop) {
        if (src == null) {
            return null;
        }
        int width = src.getWidth();
        int height = src.getHeight();
        //创建一个bitmap
        Bitmap newb = Bitmap.createBitmap(width, height, Bitmap.Config.ARGB_8888);// 创建一个新的和SRC长度宽度一样的位图
        //将该图片作为画布
        Canvas canvas = new Canvas(newb);
        //在画布 0，0坐标上开始绘制原始图片
        canvas.drawBitmap(src, 0, 0, null);
        //在画布上绘制水印图片
        canvas.drawBitmap(watermark, paddingLeft, paddingTop, null);
        // 保存
        canvas.save(Canvas.ALL_SAVE_FLAG);
        // 存储
        canvas.restore();
        return newb;
    }

绘制后的效果应该是这样的，欢迎拍砖~

![给图片加上水印.png](http://upload-images.jianshu.io/upload_images/817079-e31e704b7d9bc362.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

附上源代码：[github传送门](https://github.com/echoMu/WaterMaskDemo)