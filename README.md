# 陳鍾誠的課程

欄位       |  內容
----------|----------------------------
課程名稱   | 軟體工程
開課單位   | 金門大學資訊工程系
開課年度   | 106 年上學期
學員姓名   | 周明萱
教師姓名   | 陳鍾誠
上課教材   | [wiki](https://github.com/cccnqu/se106a/wiki)
程式範例   | [example](example)
練習作業   | [exercise](exercise)
專案作品   | [project](project)
學習筆記   | [wiki](../../wiki)
疑問解答   | [issue](https://github.com/cccnqu/se106a/issues)

// 導入koa
const Koa = require('koa');
const router = require('koa-router')()
const koaStatic = require('koa-static')
const app = module.exports = new Koa();
app.use(router.routes())
app.use(koaStatic('./public', {index:'index.html'}))

// 連結firestore
const admin = require('firebase-admin');
const serviceAccount = require('./DBkey.json');
admin.initializeApp({
  credential: admin.credential.cert(serviceAccount),
  databaseURL: 'https://stumanager-a4e0e.firebaseio.com'
});

const db = admin.firestore();
db.settings({ timestampsInSnapshots: true });

// 導入控制樹梅派GPIO
const Gpio = require('onoff').Gpio;

// 接收請求
router.get('/qrkey', async(ctx, next) => {
    // 印出URL後面的字串 console.log(ctx.request.querystring)
    
        // 查詢出對應的led_No
        db.collection('Mailroom').doc(ctx.request.querystring)
          .get().then(doc => {
          if (!doc.exists) {
            console.log('無結果');
          } else {
            console.log('led號碼:', doc.data().led_No); 
            // 
            startLED(doc.data().led_No)
          }
        })
        .catch(err => {
          console.log('Error', err);
        });
})

function startLED(number) { 
    // Gpio 通電 
    const LED = new Gpio(number, 'out');
    LED.writeSync(1); // Turn LED on
}

function endLED() { 
  //Gpio 斷電
  LED.writeSync(0); // Turn LED off
  LED.unexport(); // 釋放Gpio資源
}

setTimeout(endLED, 5000); // 5秒之後斷電

// 設定server位址
if (!module.parent){
    app.listen(3000)
    console.log("由此進入: http://localhost:3000")
} 

