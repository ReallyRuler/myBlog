# 微信小游戏Api应用案例
1、获取自身信息（主域）

  wx.createUserInfoButton()

  由于微信取消了wx.getUserInfo()，其调用结果会默认失败（仅开发环境），现在使用wx.createUserInfoButton()生成一个获取用户信息权限的按钮，其官方api在https://developers.weixin.qq.com/minigame/dev/api/open-api/user-info/wx.createUserInfoButton.html ，其官方案例
  
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
