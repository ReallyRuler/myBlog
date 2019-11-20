#微信小游戏Api应用案例
1、获取自身信息（主域）

  wx.createUserInfoButton()

  由于微信取消了wx.getUserInfo()，其调用结果会默认失败（仅开发环境），现在使用wx.createUserInfoButton()生成一个获取用户信息权限的按钮，其官方ap在https://developers.weixin.qq.com/minigame/dev/api/open-api/user-info/wx.createUserInfoButton.html
