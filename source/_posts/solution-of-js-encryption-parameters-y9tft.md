<p>基本上所有前端加密都是用的CryptoJS库，因此可以通过特征函数定位加密函数。比如CryptoJS要求加密时需要对key（密钥）进行解析(parse)后才能作为参数，因此所有需要加密的地方均会调用<code>enc.Utf8.parse</code>函数，比如</p>
<p>​<img src="https://raw.githubusercontent.com/lovelyjuice/picgo/master/img202309192021786.png" alt="" />​</p>
<p>即便是webpack打包的前端代码，这个函数名也不会被混淆为abcd之类的名字，所以搜索这个函数名可以定位到加密的地方，然后打断点，在调试器中看到密钥的值。</p>
<p>‍</p>
