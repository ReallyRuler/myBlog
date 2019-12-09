# 微信小游戏Api应用案例
1、获取自身信息（主域）
  wx.createUserInfoButton()
  
  由于微信取消了wx.getUserInfo()，其调用结果会默认失败（仅开发环境），现在使用wx.createUserInfoButton()生成一个获取用户信息权限的按钮，其官方api在https://developers.weixin.qq.com/minigame/dev/api/open-api/user-info/wx.createUserInfoButton.html ，其官方案例为 
  
 ```官方demo
 let button = wx.createUserInfoButton({
  type: 'text',
  text: '获取用户信息',
  style: {
    left: 10,
    top: 76,
    width: 200,
    height: 40,
    lineHeight: 40,
    backgroundColor: '#ff0000',
    color: '#ffffff',
    textAlign: 'center',
    fontSize: 16,
    borderRadius: 4
  }
})
button.onTap((res) => {
  console.log(res)
})
```

其效果为创建一个红色背景按钮，点击时会弹出申请用户权限，实际上在Cocos Creator上对于这种api使用方式比较别扭，查阅论坛发现一种十分方便的解决办法https://forum.cocos.org/t/wx-createuserinfobutton/60207/9 ，下面的代码在我目前程序中使用

```结合Cocos的方法
public static createAuthorizeBtn(btnNode:cc.Node,callBackFun) {
        let btnSize = cc.size(btnNode.width+10,btnNode.height+10);
        let frameSize = cc.view.getFrameSize();
        let winSize = cc.director.getWinSize();
        //适配不同机型来创建微信授权按钮
        let left = (winSize.width*0.5+btnNode.x-btnSize.width*0.5)/winSize.width*frameSize.width;
        let top = (winSize.height*0.5-btnNode.y-btnSize.height*0.5)/winSize.height*frameSize.height;
        let width = btnSize.width/winSize.width*frameSize.width;
        let height = btnSize.height/winSize.height*frameSize.height;

        let self = this;
        self.btnAuthorize = wx.createUserInfoButton({
            type: 'text',
            text: '',
            style: {
                left: left,
                top: top,
                width: width,
                height: height,
                lineHeight: 0,
                backgroundColor: '',
                color: '#ffffff',
                textAlign: 'center',
                fontSize: 16,
                borderRadius: 4
            }
        })
        
        self.btnAuthorize.onTap((uinfo) => {
            console.log("onTap uinfo: ",uinfo);
            if (uinfo.userInfo) {
                console.log("wxLogin auth success");
                wx.showToast({title:"授权成功"});
                this.userInfo = uinfo.userInfo
                this.nickName = this.userInfo.nickName
                this.avatarUrl = this.userInfo.avatarUrl
                this.gender = this.userInfo.gender

                console.log("个人信息"+this.userInfo);
                callBackFun();
            }else {
                console.log("wxLogin auth fail");
                wx.showToast({title:"授权失败"});
            }
        });
    }
```

其效果是创建一个透明获取用户权限的按钮，将其与Cocos Creator中的按钮绑定，点击响应直接授权成功，并在授权成功后调用自定义的回调函数。

目前发现存在问题，生成的透明按钮在Cocos Creator场景跳转后仍然存在，需要手动删除，在btnAuthorize.distroy()可以摧毁按钮，btnAuthorize.hide()可以隐藏按钮，使点击响应失效。
.
2、显示微信头像

根据获取到的微信头像avatarUrl，可以将头像显示出来，目前我程序中的方法是

```显示微信头像
public static createImage(avatarUrl,photoSprite) {
        console.log("avatarUrl="+avatarUrl)
        try {
            let image = wx.createImage();
            image.onload = () => {
                try {
                    let texture = new cc.Texture2D();
                    texture.initWithElement(image);
                    texture.handleLoadedTexture();
                    photoSprite.spriteFrame = new cc.SpriteFrame(texture);
                } catch (e) {
                    cc.log(e);
                    photoSprite.node.active = false;
                }
            };
            image.src = avatarUrl;
        }catch (e) {
            cc.log(e);
            photoSprite.node.active = false;
        }
    }
```

将url对应的图片赋予输入节点的cc.Sprite组件

3、主动分享与通过分享传参
可以通过发起一次主动分享来向被分享人传递参数，一般用于邀请进入同一房间游戏，下面的是发起分享的代码
```主动分享
public static wxShare(roomid,avatarUrl){
        console.log("开始分享")
        var self = this;
        cc.loader.loadRes("texture/share",function(err,data){
            console.log("读取资源成功 data.url="+data.url+" roomid="+roomid);
            cc.director.loadScene('Share');
            wx.shareAppMessage({
                title: "一起玩游戏！",
                imageUrl: data.url,
                query: "roomid="+roomid+"&avatarUrl="+avatarUrl
            },)
        });
    }
  ```
点击分享链接进入的用户可以通过wx.getLaunchOptionsSync()获取传递的参数，可以放到游戏开始场景的onload()方法中，来区别对待自己进入还是通过他人分享链接进入游戏
```获取参数
let launch = wx.getLaunchOptionsSync();
        console.log(launch);  
        if(launch.query != undefined && launch.query != null && launch.query.length != 0){
            if(launch.query.roomid != undefined && launch.query.avatarUrl != undefined){
                /**一些操作*/、
            }
        }
```
