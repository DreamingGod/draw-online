<template>
    <div style="width: 100%;height: 100%">
        <div>
            <h4>
                测试
            </h4>
        </div>
        <div>
            <input type="button" value="红色" @click="changecolor('red')">
            <input type="button" value="黄色" @click="changecolor('yellow')">
            <input type="button" value="绿色" @click="changecolor('green')">
            <input type="button" value="黑色" @click="changecolor('black')">
        </div>
        <div>
            <input type="button" value="粗" @click="checkpen(10)">
            <input type="button" value="中" @click="checkpen(5)">
            <input type="button" value="细" @click="checkpen(1)">
        </div>
        <div>
            <input type="button" v-model="eraserTitle" @click="checkeraser()">
        </div>
        <div>
            <input type="button" value="保存" @click="saveAsLocalImage()">
        </div>
        <img v-show="false" ref="myImg" src="../../assets/logo.png" alt="">
        <canvas id="canvas" ref="myCanvas" width="1300px" height="500px" style="border: 1px solid #c3c3c3">
            您的浏览器不支持HTML5 canvas标签
        </canvas>

    </div>
</template>

<script>
    export default {
        name: "testCanvas",
        data() {
            return {
                eraserStatus:false,
                eraserTitle:"橡皮檫",
                canvas:null,
                ctx:null,
                drawing:false,
                img:null,
                penWeight:null,//因为可能由于异步处理，所以在这里给默认值可能无效，需要在this.$nextTick里面给默认值
                penColor:null,
            }
        },
        methods:{
            draw(){
                /*let penWeight = 1;
                let penColor = '#ccc';*/
                let self=this;

                //console.log("hahah",this.ctx.lineWidth);
                this.canvas.onmousedown = function (e){
                    //console.log("开始画图")
                    let canvasBox = self.canvas.getBoundingClientRect();
                    /*let start_x = e.clientX - self.canvas.offsetLeft +document.body.scrollLeft;
                    let start_y = e.clientY - self.canvas.offsetTop + document.body.scrollTop;*/
                    let start_x = e.clientX - canvasBox.left +document.body.scrollLeft;
                    let start_y = e.clientY - canvasBox.top + document.body.scrollTop;

                    self.ctx.beginPath();
                    self.ctx.moveTo(start_x,start_y);

                    self.ctx.lineCap = 'round';
                    self.ctx.lineJoin = 'round';

                    self.ctx.strokeStyle = self.penColor;
                    self.ctx.lineWidth = self.penWeight;

                    self.canvas.onmousemove = function (e) {
                        /*let move_x = e.clientX - self.canvas.offsetLeft + document.body.scrollLeft;
                        let move_y = e.clientY - self.canvas.offsetTop + document.body.scrollTop;*/
                        let canvasBox = self.canvas.getBoundingClientRect();
                        let move_x = e.clientX - canvasBox.left +document.body.scrollLeft;
                        let move_y = e.clientY - canvasBox.top + document.body.scrollTop;
                        self.ctx.lineTo(move_x,move_y);
                        self.ctx.stroke();

                        self.canvas.onmouseup = function (e) {
                            self.ctx.closePath();
                            self.canvas.onmousemove = null;
                            self.canvas.onmouseup = null;
                        }
                    }

                    self.canvas.onmouseleave = function (e) {
                        self.ctx.closePath();
                        self.canvas.onmousemove = null;
                        self.canvas.onmouseup = null;
                    }
                }

            },
            checkpen(width){
                this.penWeight = width;
            },
            changecolor(color){
                this.penColor = color;
            },

            checkeraser(){
                this.eraserStatus = !this.eraserStatus;
                if (this.eraserStatus) {
                    this.eraserTitle = "正在使用..."
                    this.ctx.lineWidth = 20;
                    this.ctx.globalCompositeOperation = "destination-out";
                    let self=this;
                    //console.log(this.canvas.getBoundingClientRect())
                    //console.log(this.canvas.getBoundingClientRect())
                    //return false;
                    function getBoundingClientRect(x,y) {
                        var box = self.canvas.getBoundingClientRect()//获取canvas的距离浏览器视窗的上下左右距离
                        return {
                            x:x-box.left,
                            y:y-box.top
                        }
                    }

                    this.canvas.onmousedown = function (e) {
                        var first = getBoundingClientRect(e.clientX,e.clientY);
                        self.ctx.save();
                        self.ctx.beginPath();
                        self.ctx.moveTo(first.x,first.y);
                        self.drawing = true;

                    }
                    this.canvas.onmousemove = function (e) {
                        if (self.drawing) {
                            var move = getBoundingClientRect(e.clientX,e.clientY);
                            self.ctx.lineTo(move.x,move.y);
                            self.ctx.stroke();
                            self.ctx.restore();
                        }
                    }
                    this.canvas.onmouseup = function (e) {
                        self.drawing = false;
                    }

                    this.canvas.onmouseleave = function (e) {
                        self.drawing = false;
                        //self.canvas.onmouseup();
                    }
                } else {
                    this.eraserTitle = "橡皮檫"
                    this.ctx.globalCompositeOperation = "source-over";
                    this.draw();
                }
                // this.$nextTick(()=>{
                //     /*this.canvas=this.$refs.myCanvas;
                //     this.img = this.$refs.myImg;
                //     this.ctx = this.canvas.getContext("2d");
                //     this.penWeight = 1;
                //     this.penColor = 'yellow'
                //     this.draw()*/
                //
                //     this.ctx.lineWidth = 20;
                //
                // });


            },

            saveAsLocalImage ()
            {
                /*let image = this.canvas.toDataURL("image/png").replace("image/png", "image/octet-stream");
                window.location.href = image;*/
                var myCanvas = document.getElementById("canvas");
                var image = myCanvas.toDataURL("image/png").replace("image/png", "image/octet-stream");
                //window.location.href=image;
                this.downLoad(image);
            },
            downLoad(url){
                var oA = document.createElement("a");
                oA.download = 'img.png';// 设置下载的文件名，默认是'下载'
                oA.href = url;
                document.body.appendChild(oA);
                oA.click();
                oA.remove(); // 下载之后把创建的元素删除
            },



        },
        mounted(){
            console.log("挂载成功",this.canvas);
            //var canvas = this.$refs.myCanvas;
            //var ctx = canvas.getContext("2d");
            //var img = this.$refs.myImg;

            //贴图
            this.$nextTick(()=>{
                this.canvas=this.$refs.myCanvas;
                this.img = this.$refs.myImg;
                this.ctx = this.canvas.getContext("2d");
                //this.ctx.drawImage(this.img,0,0);
                //this.ctx.font="10px hengbenyinhe";
                //打水印
                //this.ctx.fillText("hengbenyinhe",160,90)
                this.penWeight = 1;
                this.penColor = 'yellow'
                this.draw()
            });

        },
    }
</script>

<style scoped>

</style>