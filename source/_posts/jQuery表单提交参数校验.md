---
title: jQuery表单提交参数校验
date: 2019-08-04 21:57:25
updated: 2019-08-04 21:57:25
tags: 
- javascript
- jQuery
---

### 简介

最近写web项目比较多，发现一个比较好用的表单校验的js，遂记录之。
<!--more-->
### 关键步骤

1. input标签中只需加入

```
<input type="text" maxlength="20" class="txt03 f-r3 required" data-valid="isNonEmpty||between:3-20||isUname" data-error="用户名不能为空||用户名长度3-20位||只能输入中文、字母、数字、下划线，且以中文或字母开头" />
```
`data-valid`和`data-error`两个参数，样式中加入`required`。

2. javascrpit校验的时候只需调用

```
if(!verifyCheck._click()) return;
```
那么输入的值被校验通过了，`verifyCheck._click()`函数就会返回true，而且校验规则可以自定义，支持正则，简单快捷。
### 具体实现

1. 完整html代码如下

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<form>
    <div class="f-fl item-ifo" id="verifyCheck">
        <input type="text" maxlength="20" class="txt03 f-r3 required" tabindex="1" data-valid="isNonEmpty||between:3-20||isUname" data-error="用户名不能为空||用户名长度3-20位||只能输入中文、字母、数字、下划线，且以中文或字母开头" id="adminNo" />
    </div>
    <button id="btn_part1">确定</button>
</form>
</body>
<script type="text/javascript" src="https://demo.demohuo.top/jquery/23/2301/demo/js/jquery-1.11.1.min.js"></script>
<script>
    $(function(){
        //第一页的确定按钮
        $("#btn_part1").click(function(){
            debugger;
            if(!verifyCheck._click()) return;
            // 业务代码
        });
    });

</script>
<script type="text/javascript" src="verify.js"></script>
</html>
```
2. 关键的verify.js代码

```javascript
/**
 * 功能说明:		输入验证
 * 使用方法:
 * <input class="required" type="text" data-valid="isNonEmpty||isEmail" data-error="email不能为空||邮箱格式不正确" id="" />
 * 1、需要验证的元素都加上【required】样式
 * 2、@data-valid		验证规则，验证多个规则中间用【||】隔开，更多验证规则，看rules和rule，后面遇到可继续增加
 * 3、@data-error		规则对应的提示信息，一一对应
 *
 * @js调用方法：
 * verifyCheck({
 *  	formId:'verifyCheck',		<验证formId内class为required的元素
 *	onBlur:null,				<被验证元素失去焦点的回调函数>
 *	onFocus:null,				<被验证元素获得焦点的回调函数>
 *	onChange: null,				<被验证元值改变的回调函数>
 *	successTip: true,			<验证通过是否提示>
 *	resultTips:null,			<显示提示的方法，参数obj[当前元素],isRight[是否正确提示],value[提示信息]>
 *	clearTips:null,				<清除提示的方法，参数obj[当前元素]>
 *	code:true					<是否需要手机号码输入控制验证码及点击验证码倒计时,目前固定手机号码ID为phone,验证码两个标签id分别为time_box，resend,填写验证框id为code>
 *	phone:true					<改变手机号时是否控制验证码>
 * })
 * $("#submit-botton").click(function(){		<点击提交按钮时验证>
 *  	if(!common.verify.btnClick()) return false;
 * })
 *
 */
(function ($) {
    var timerT,timerC=60,opt;
    var verifyCheck=function(config){
        config=$.extend(require.defaults,config||{});
        opt=config;
        return (new require())._init(config);
    };
    function require(options){
        var rule={//验证规则
            phone:/^1(3\d|5[0-35-9]|8[025-9]|47)\d{8}$/,
            company:/^[\u4E00-\u9FA5a-zA-Z][\u4E00-\u9FA5a-zA-Z0-9\s-,-.]*$/,
            uname:/^[\u4E00-\u9FA5a-zA-Z][\u4E00-\u9FA5a-zA-Z0-9_]*$/,
            zh:/^[\u4e00-\u9fa5]+$/,//纯中文
            card:/^((1[1-5])|(2[1-3])|(3[1-7])|(4[1-6])|(5[0-4])|(6[1-5])|71|(8[12])|91)\d{4}(((((19|20)((\d{2}(0[13-9]|1[012])(0[1-9]|[12]\d|30))|(\d{2}(0[13578]|1[02])31)|(\d{2}02(0[1-9]|1\d|2[0-8]))|(([13579][26]|[2468][048]|0[48])0229)))|20000229)\d{3}(\d|X|x))|(((\d{2}(0[13-9]|1[012])(0[1-9]|[12]\d|30))|(\d{2}(0[13578]|1[02])31)|(\d{2}02(0[1-9]|1\d|2[0-8]))|(([13579][26]|[2468][048]|0[48])0229))\d{3}))$/, //身份证号
            int:/^[0-9]*$/,
            s:''
        };
        this.rules={
            isNonEmpty: function(value, errorMsg) {//不能为空
                errorMsg=errorMsg||" ";
                if (!value.length) return errorMsg;
            },
            minLength: function(value, length, errorMsg) { //大于
                errorMsg=errorMsg||" ";
                if (value.length < length) return errorMsg;
            },
            maxLength: function(value, length, errorMsg) {//小于
                errorMsg=errorMsg||" ";
                if (value.length > length) return errorMsg;
            },
            isRepeat:function(value, range, errorMsg){ //重复密码
                errorMsg=errorMsg||" ";
                if(value!==$("#"+range).val()) return errorMsg;
            },
            between: function(value, range, errorMsg) {//大于小于
                errorMsg=errorMsg||" ";
                var min = parseInt(range.split('-')[0]);
                var max = parseInt(range.split('-')[1]);
                if (value.length < min || value.length > max) return errorMsg;
            },
            level:function(value,range,errorMsg){//密码复杂程度
                errorMsg=errorMsg||" ";
                var r=verifyCheck.pwdStrong(value);
                if(range>4) range=3;
                if(r<range)  return errorMsg;
            },
            isPhone: function(value, errorMsg) {
                errorMsg=errorMsg||" ";
                if (!rule.phone.test(value)) return errorMsg;
            },
            isCompany:function(value, errorMsg) {
                errorMsg=errorMsg||" ";
                if (!rule.company.test(value)) return errorMsg;
            },
            isInt: function(value, errorMsg) {
                errorMsg=errorMsg||" ";
                if (!rule.int.test(value)) return errorMsg;
            },
            isUname: function(value, errorMsg) {
                errorMsg=errorMsg||" ";
                if (!rule.uname.test(value)) return errorMsg;
            },
            isZh: function(value, errorMsg) {
                errorMsg=errorMsg||" ";
                if (!rule.zh.test(value)) return errorMsg;
            },
            isCard: function(value, errorMsg) {
                errorMsg=errorMsg||" ";
                if (!rule.card.test(value)) return errorMsg;
            },
            isChecked: function(value, errorMsg, el){
                errorMsg=errorMsg||" ";
                var a=$(el).find('input:checked').length,
                    b=$(el).find('.on').length;
                if(!a && !b) return errorMsg;
            }
        }
    };
    require.prototype={
        _init:function(config){
            this.config=config;
            this.getInputs = $('#'+config.formId).find('.required:visible');
            var result=false;
            var $this=this;
            if(config.code){
                $("#verifyYz").click(function(){
                    $("#time_box").text("60 s后可重发");
                    $this._sendVerify()
                });
            }
            $('body').on({
                blur:function(event){
                    $this.formValidator($(this));
                    if(config.phone && $(this).attr("id")==="phone") $this._change($(this));
                    config.onBlur ? config.onBlur($(this)) : '';
                },
                focus:function(event){
                    config.onFocus ? config.onFocus($(this)) : $(this).parent().find("label.focus").not(".valid").removeClass("hide").siblings(".valid").addClass("hide") && $(this).parent().find(".blank").addClass("hide") && $(this).parent().find(".close").addClass("hide");
                },
                keyup:function(event){
                    if(config.phone && $(this).attr("id")==="phone") $this._change($(this));
                },
                change:function(event){
                    config.onChange ? config.onChange($(this)) : '';
                }
            },"#"+config.formId+" .required:visible");
            $('body').on("click",".close",function(){
                var p=$(this).parent(),
                    input=p.find("input");
                input.val("").focus();
            });
        },
        /**
         * 功能说明:		分割要验证的规则并整理
         * @author:		vivy <lizhizyan@qq.com>
         * @time:		2015-9-22 17:51:30
         * @version:	V1.1.0
         * @param:		$el<当前被验证的元素>
         */
        formValidator:function($el){
            var dataValid = $el.attr('data-valid');
            if(dataValid===undefined) return false;
            var validLen = dataValid.split('||');
            var errCollection = $el.attr('data-error');
            if(errCollection===undefined) errCollection="";
            var errMsgAry = errCollection.split("||");
            var ruleAry = [];
            for (var i = 0; i < validLen.length; i++) {
                ruleAry.push({
                    strategy: validLen[i],
                    errorMsg: errMsgAry[i]
                });
            };
            return this._add($el, ruleAry);
        },
        /**
         * 功能说明:		获取验证结果
         * @author:		vivy <lizhizyan@qq.com>
         * @time:		2015-9-22 17:53:30
         * @version:	V1.1.0
         * @param:		dom<当前被验证的元素>
         * @param:		rules<逐一验证的数组>
         */
        _add:function(dom,rules){//验证
            var self = this;
            for (var i = 0, rule; rule = rules[i++];) {
                var strategyAry = rule.strategy.split(':');
                var errorMsg = rule.errorMsg;
                var strategy = strategyAry.shift(); // 前删匹配方式并赋值
                strategyAry.unshift(dom.val()); // 前插value值
                strategyAry.push(errorMsg); // 后插出错提示
                strategyAry.push(dom);
                var c=self.rules[strategy].apply(dom,strategyAry);
                if(c){
                    opt.resultTips ? opt.resultTips(dom,false,c) : verifyCheck._resultTips(dom,false,c);
                    return false;
                }
            }
            opt.successTip ? (
                opt.resultTips ? opt.resultTips(dom,true) : verifyCheck._resultTips(dom,true)
            ) : verifyCheck._clearTips(dom);
            return true;
        },
        /**
         * 功能说明:		发送验证码
         * @author:		vivy <lizhizyan@qq.com>
         * @time:		2015-9-22 17:49:30
         * @version:	V1.1.0
         * @param:		resend<发送验证码的元素ID>
         * @param:		time_box<显示计时器的元素ID>
         * @update:		zhanghx13855 2015-09-24 14:58:30 手机号码更改发送验证码按钮在倒计时没有为0时不出现
         */
        _sendVerify:function(){
            var that=this;
            $("#verifyYz").text("发送验证码").hide();
            $("#time_box").text("10 s后可重发").show();
            if(timerC===0){
                clearTimeout(timerT);
                timerC=60;
                var re=/^1([^01269])\d{9}$/;
                if(!re.test($("#phone").val())){
                    $("#time_box").text("发送验证码");
                }else{
                    $("#verifyYz").show();
                    $("#time_box").hide();
                }
                return;
            }
            $("#time_box").text(timerC+" s后可重发");
            timerC--;
            timerT=setTimeout(function(){that._sendVerify();},1000);
        },
        /**
         * 功能说明:		更改手机号对验证码的控制
         * @author:		vivy <lizhizyan@qq.com>
         * @time:		2015-9-22 17:51:30
         * @version:	V1.1.0
         * @param:		$this<即$(this)>
         * @param:		btnId<提交的按钮ID>
         * @param:		resend<发送验证码的元素ID>
         * @param:		time_box<显示计时器的元素ID>
         */
        _change:function($this){
            var ts=this;
            if($this.val().length!=11)
            {
                $("#verifyYz").hide();
                $("#time_box").show();
                if(timerC===60) $("#time_box").text("发送验证码");
                $("#verifyNo").val("");
                this.config.clearTips?this.config.clearTips($("#verifyNo")):verifyCheck._clearTips($("#verifyNo"));
                return;
            }
            var re=/^1([^01269])\d{9}$/;
            if(!re.test($this.val())) return false;
            if(timerC===60)
            {
                $("#verifyYz").show();
                $("#time_box").hide();
            }
            else
            {
                $("#verifyYz").hide();
                $("#time_box").show();
            }
        }
    };

    /***************************外部可调用的公用方法************************
     /**
     * 功能说明:		外部触发验证结果，常用于点击提交按钮时调用，返回验证结果
     * @author:		vivy <lizhizyan@qq.com>
     * @time:		2015-9-25 15:37:30
     * @version:		V1.1.0
     * @update:
     * @param:		formId<被验证的元素包ID>
     * @return:		验证的结果
     */
    verifyCheck._click=function(formId){
        debugger;
        formId = formId || opt.formId;
        var obj=$("#"+formId).find('.required:visible'),
            self=this,result=true,t=new require(),r=[];

        $.each(obj, function(index, el){
            result=t.formValidator($(el));
            if(result) r.push(result);
        });

        if(obj.length!==r.length) result=false;
        return result;
    };
    /**
     * 功能说明:		清空提示
     * @author:		vivy <lizhizyan@qq.com>
     * @time:		2015-9-22 17:55:30
     * @version:	V1.1.0
     * @param:		obj<当前被验证的元素>
     */
    verifyCheck._clearTips=function(obj){
        obj.parent().find(".blank").addClass("hide");
        obj.parent().find(".valid").addClass("hide");
        obj.removeClass("v_error");
    };
    /**
     * 功能说明:		新增提示
     * @author:		vivy <lizhizyan@qq.com>
     * @time:		2015-9-22 17:57:30
     * @version:	V1.1.0
     * @param:		obj<当前被验证的元素>
     * @param:		isRight<是否是正确的提示>
     * @param:		value<提示信息>
     */
    verifyCheck._resultTips=function(obj,isRight,value){
        obj.parent().find("label.focus").not(".valid").addClass("hide").siblings(".focus").removeClass("hide");
        obj.parent().find(".close").addClass("hide");
        obj.removeClass("v_error");
        value=value||"";
        if(value.length>21) value="<span>"+value+"</span>";
        var o=obj.parent().find("label.valid");
        if(!isRight){
            o.addClass("error");
            obj.addClass("v_error");
            if($.trim(obj.val()).length>0)
                obj.parent().find(".close").removeClass("hide");
        }else{
            obj.parent().find(".blank").removeClass("hide")
        }
        o.text("").append(value);
    };
    /**
     * 功能说明:		获取字符串所有中文的字节数(一个汉字=2个字节)
     * @author:		vivy <lizhizyan@qq.com>
     * @time:		2015-8-31 15:37:30
     * @version:		V1.1.0
     * @update:
     * @param:		tips<提示的内容>
     * @param:		timeout<提示停留的时间，可不配置，默认为2秒>
     */
    verifyCheck.textChineseLength=function(str){
        var re=/[\u4E00-\u9FA5]|[\u3001-\u3002]|[\uFF1A-\uFF1F]|[\u300A-\u300F]|[\u3010-\u3015]|[\u2013-\u201D]|[\uFF01-\uFF0E]|[\u3008-\u3009]|[\u2026]|[\uffe5]/g;
        if(re.test(str))        //使用正则判断是否存在中文
            return str.match(re).length; //返回中文的个数
        else
            return 0;
    };
    /**
     * 功能说明:		判断密码强度
     * @author:		vivy <lizhizyan@qq.com>
     * @time:		2015-9-23 10:11:30
     * @version:	V1.1.0
     * @param:		val<密码值>
     */
    verifyCheck.pwdStrong=function(val){
        var lv=0;
        if(val.match(/[a-z]/g)){lv++;}
        if(val.match(/[A-Z]/g)){lv++;}
        if(val.match(/[0-9]/g)){lv++;}
        if(val.match(/(.[^a-z0-9A-Z])/g)){lv++;}
        if(lv > 4){lv=4;}
        if(lv===0) return false;
        return lv;
    };
    /**
     * 默认配置
     */
    require.defaults = {
        formId:'verifyCheck',	//验证的ID
        onBlur:null,//被验证元素失去焦点的回调函数
        onFocus:null,//被验证元素获得焦点的回调函数
        onChange: null,//被验证元素值改变的回调函数
        successTip: true,//验证通过是否有提示
        resultTips:null,//验证提示的回调函数，传回参数obj[当前元素],isRighe[验证是否通过],value[提示的值]
        clearTips:null,//清空提示的回调函数，传回参数obj[当前元素]
        code:true,//是否需要点击获取验证码倒计时
        phone:false//是否需要手机号码输入控制验证码的显示
    };
    window.verifyCheck = $.verifyCheck= verifyCheck;
})(jQuery);
```
