History
==========

## 1.0.0 / 2015-03-14

*  fork of aralejs/base 1.2.0,
*  增加_onChange实现;当所有任意abc属性改变时,先调用实例的_onChangeAbc,然后先调用实例的_onChange,如果_onChangeAbc返回值为false,则不调用实例的_onChange;