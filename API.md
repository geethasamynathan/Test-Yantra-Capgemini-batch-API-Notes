Http Protocol

Http Verbs (httpGet,HttpPost,HttpPut,HttpPatch,HttpDelete)

httpGet ==> To read (or) fetch data using api
httpPost ==>  Add (or) create new item 
httpPut ==>update the record
HttpPatch ==> PartialUpdate
HttpDelete ==> Delete (or)Remove data

Http Status code 
=================

200 ==> OK
201 ==> Created
401==> unauthorized
403 ==> Forbidden
500 ==> internal server Error


web API core ==> by default it return json data.  If we need to checge the 
response type in another format (XML,Text) in that situation we need specify the mediaType Formatter  (or)  content type (application/xml, application/json)

