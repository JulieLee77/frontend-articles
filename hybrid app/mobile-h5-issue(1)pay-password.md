#移动端h5问题探索(1)支付密码输入框
**主要涉及问题**

1. `input[type=tel]`与`input[type=number]`
2. `-webkit-text-security` mobile延迟
3. ios `focus()`方法问题

**需求描述**

点击页面的按钮时发出请求，根据请求结果决定是否显示支付密码输入弹框（聚焦输入框调出数字键盘）。

##思路解读
1. ###input[type=tel]与input[type=number]
	因为需要调出数字键盘，就排除了 `input[type=password]`，而 `input[type=tel]`或 `input[type=number]`均可调出数字键盘，但 `input[type=number]`可输入负数和小数点（且还有spin-button）。
	
	考虑到 `-webkit-text-security`在mobile的延迟，故需隐藏input，用元素显示圆点模拟（这样也方便调整样式）。

	**HTML密码部分**

	```
	<input type="tel" v-model="pwd" class="pwdInput" maxlength="6" @keydown="pwdKeyDown($event)" v-el:pwd v-if="lockPwd||!planed">
	<section class="mask pwd" v-show="inputPwd">
	  <div class="modal">
	    <div class="modal-content">
	      <h4>请输入6位数支付密码</h4>
	      <div class="pwd-box">
	        <ul @click.prevent="focusPwd()"><li v-for="item in 6" track-by="$index"><span v-if="pwd.length>$index"></span></li></ul>
	      </div>
	    </div>
	    <div class="footer clearfix">
	      <div><input type="button" value="取消" @click="inputPwd=false"/></div>
	      <div><input type="button" value="确定" :disabled="pwd===null||pwd===undefined||pwd.length<pwdLength" @click="submitPlan()"/></div>
	    </div>
	  </div>
	</section>
	```
	由于在显示弹出框时就必须focus输入框，而不能有timeout或异步请求，因此input输入框应一直在DOM中，显示在页面视线外。
	
	**css密码部分**
	
	```
	//支付密码
	input.pwdInput[type=tel]
	  position fixed
	  top 50%
	  left -100%
	  padding 0
	  border none
	  color transparent
	  background transparent
	  &:focus
	    outline none
	
	.pwd .modal
	  .pwd-box
	    width 100%
	    margin 0 auto
	    padding-top gap-12
	    text-align center
	    @extend .border-top-1px
	    ul
	      li
	        box-sizing border-box
	        display inline-block
	        width 16.667%
	        border-top 1px solid gray-4
	        border-right 1px solid gray-4
	        border-bottom 1px solid gray-4
	        &:after
	          content ''
	          display inline-block
	          vertical-align middle
	          width 0
	          height 100%
	        span
	          display inline-block
	          vertical-align middle
	          text-align center
	          width 10px
	          height 10px
	          border-radius 999px
	          background gray-8
	        &:not(:first-child)
	          border-left none
	        &:first-child
	          border-left 1px solid gray-4
	          border-top-left-radius 2px
	          border-bottom-left-radius 2px
	        &:last-child
	          border-top-right-radius 2px
	          border-bottom-right-radius 2px	
	```


2. ###event.returnValue=false;
	在keydown方法中禁止键盘输入
	
	**密码输入keydown监听方法**
	
	```
	pwdKeyDown: function (event) {
	      var returnVal = true;
	      if (event.keyCode >= 48 && event.keyCode <= 57) { //0-9
	        (this.pwd !== null && this.pwd !== undefined && this.pwd.length == this.pwdLength) && (returnVal = false); //密码已输完,禁止继续输入
	      } else if (event.keyCode === 13) {//enter
	        this.pwd !== null && this.pwd !== undefined && this.pwd.length === this.pwdLength && this.submitPlan();//若已输完6位,则enter提交
	      } else if (event.keyCode !== 8 && event.keyCode !== 46) {//非 delete & backspace
	        returnVal = false;
	      }
	      event.returnValue = returnVal;
	    }
	```
3. ###关于document.activeElement.blur();
	`document.activeElement`可获取当前聚焦元素。

	本来是为了解决ios自动focus()问题，但直接调用javascript的focus()方法后，document.activeElement获取到新设置的元素，并没有调出键盘。这应该是ios的bug，查找到的方法是，在click方法中立即focus()， 不能有在timeout／异步请求等延时之后。
4. ###ios自动focus()实现
	需要在click方法中触发，若需根据HTTP请求后的结果判断是否要focus()，则需改用同步请求。同时，不能在`timeout`方法后focus()，原理同异步请求。
	
	**判断显示输入密码弹出框函数**
	
	```
	showInputPwd: function () {
	      if (this.lastCode === 2013) {
	        this.lockPwd = true;
	        return;
	      }
	      var _this = this;
	      $.ajax({
	          url: 'js/0002.json',
	          method: 'GET',
	          async: false//solve ios focus() bug
	        })
	        .done(function (data) {
	          if (data.code === 2013) {//支付密码已锁定
	            _this.lockPwd = true;
	            _this.lastCode = data.code;
	          } else if (data.code === 0) {
	            _this.inputPwd = true;
	            _this.$set('pwd', null);
	            setTimeout(function () {
	              if (!_this.list) {
	                _this.list = document.querySelectorAll('.pwd-box li');
	                var width = document.querySelector('.pwd-box').clientWidth / 6;
	                for (var i = 0, v; v = _this.list[i]; i++) {
	                  v.style.height = width + 'px';
	                }
	              }
	            }, 100);
	           _this.$els.pwd.focus();
	          }
	        })
	        .fail(function (error) {
	          console.log('fail' + ',' + error);
	        });
	    }
	```
	
#实现方法

使用了vue和jquery（主要用它的同步请求方法，若不需要改用vue-resource插件）[demo](https://julielee77.github.io/demo/0002.html)

