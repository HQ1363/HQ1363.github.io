
# go http请求和CURL返回结果不一致情况
go 代码发送 http 请求时，会自动默认添加 Accept-Encoding: gzip ，可能引发和CURL请求不一致的地方
