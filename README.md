# MeiTuanCinemaSmartChoose
模拟美团app推荐座位算法.
# 使用
clone本项目，然后npm install后直接npm start启动开发服务器即可
# 简介
![操作界面](https://github.com/houzisbw/MeiTuanCinemaSmartChoose/blob/master/img/1.png)
# 算法流程
(1)推荐算法首先从影院中间排数的后一排的正中间开始搜索<br>
(2)优先向后排方向进行搜索，后排搜索完成后再从中间起始位置向前排搜索 <br>
(3)后排搜索完成后，每一行都会有一个结果(每一行的结果是最靠近中轴线的那一组座位)，取这些结果中距离中轴线最小的那个结果作为最终结果，而不是距离屏幕越近的<br>
(4)只考虑并排且连续的座位，不能不在一排或者一排中间有分隔，比如过道之类的
(5)一组选定的座位左右2侧不能留下单个空座位




座位预览

模板

      seat_Model_Config : {
         modelName:'', // 模板名称
         seatModelType:'',//座位模型('u-shape','gyration-shape','desk-shape,'customize')
         seatColNum: 7 ,//横向座位数
         seatRowNum:7,//纵向座位数:
         orientation:'u',//朝向（上下左右 一次 u d l r）默认朝上
         circleNum:3, // 有多少环，最多支持三环
         infrastructures:[],//设备信息 infrastructureInfo
         seats:[],// 座位信息
       },
       // 设施信息
       infrastructureInfo:{
           topIndex:0 ,// 设施顶部位于布局的第几格
           leftIndex:0 ,// 设施左边部分位于布局的第几格
           infrastructureWidth:0, //设施横向占多少格
           infrastructureHeight:0,//设施纵向占多少格
           infrastructureName: '',   // 设施名称
           infrastructureOrientation:'' //设施朝（上下左右 依次 u d l r）默认朝上
        },
        // 座位信息
        seatInfo:{
           seatType:'',//座位类型（普通座位，主席座等）
           rowIndex:0, //座位位于布局的第几行
           colIndex:0, //座位位于布局的第几列
           seatNo:'', //座位编号
           seatLable:'' ,//座位文字显示信息（与座位号二者只能显示其中一个）
           seatOrientation:''//座位朝向（上下左右 依次 u d l r）默认朝上
        },




