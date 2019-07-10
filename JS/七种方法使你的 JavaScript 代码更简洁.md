## 前言

  同样一个功能，可以用很多方法来实现，有时候由于项目时间紧张，导致很多时候只是实现了功能，往往忽视了代码质量。下面几种代码重构方法，以便大家可以写出更漂亮的代码。本文内容是我的读书笔记，摘自《Javascript设计模式与开发》。

### 一，提炼函数

    var getUserInfo = function(){
      ajax( 'http:// xxx.com/userInfo', function( data ){
          console.log( 'userId: ' + data.userId );
          console.log( 'userName: ' + data.userName );
          console.log( 'nickName: ' + data.nickName );
      })
    })

    //  重构后
    var getUserInfo = function(){
      ajax( 'http:// xxx.com/userInfo', function( data ){
        printDetails( data ); });
    };

    var printDetails = function( data ){ 
      console.log( 'userId: ' + data.userId ); 
      console.log( 'userName: ' + data.userName ); 
      console.log( 'nickName: ' + data.nickName );
    };
  

### 二，合并重复的条件片段

    var paging = function( currPage ){
      if ( currPage <= 0 ){
        currPage = 0;
        jump( currPage );
      }else if ( currPage >= totalPage ){
        currPage = totalPage;
        jump( currPage );
      }else{
        jump( currPage );
      }
    }

    // 重构后，把重复函数独立出来
    var paging = function( currPage ){
        if ( currPage <= 0 ){
            currPage = 0;
        }else if ( currPage >= totalPage ){
            currPage = totalPage; 
        }
        jump( currPage );
    };

### 三，把条件分支语句提炼成函数

    var getPrice = function( price ){
      var date = new Date();
      if ( date.getMonth() >= 6 && date.getMonth() <= 9 ){
        return price * 0.8;
      }
      return price; 
    };

    // 重构后，改成能够理解的函数
    var isSummer = function(){
      var date = new Date();
      return date.getMonth() >= 6 && date.getMonth() <= 9;
    };

    var getPrice = function( price ){ 
      if ( isSummer() ){ 
      return price * 0.8;
    }
      return price; 
    };

### 四，合理使用循环

    var createXHR = function(){ 
      var xhr;
      try{
        xhr = new ActiveXObject( 'MSXML2.XMLHttp.6.0' );
      }catch(e){ 
        try{
          xhr = new ActiveXObject( 'MSXML2.XMLHttp.3.0' ); 
        }catch(e){
          xhr = new ActiveXObject( 'MSXML2.XMLHttp' ); 
        }    
      }
      return xhr; 
    };
    var xhr = createXHR();

    // 重构后，使用遍历更简洁
    var createXHR = function(){
      var versions= [ 'MSXML2.XMLHttp.6.0ddd', 'MSXML2.XMLHttp.3.0', 'MSXML2.XMLHttp' ];
      for ( var i = 0, version; version = versions[ i++ ]; ){
        try{
          return new ActiveXObject( version );
        }catch(e){
          // ...
        }
      }
    };
    var xhr = createXHR();

### 五，提前让函数退出代替嵌套条件分支

    var del = function( obj ){
      var ret;
      if ( !obj.isReadOnly ){   
        if ( obj.isFolder ){ 
          ret = deleteFolder( obj );
        }else if ( obj.isFile ){
          ret = deleteFile( obj );
        }
      }
      return ret;
    }

    // 函数只有一个出口，改成多个出口
    var del = function( obj ){
      if ( obj.isReadOnly ){
        return;     //反转 if 表达式
      }
      if ( obj.isFolder ){
        return deleteFolder( obj );
      }
      if ( obj.isFile ){
        return deleteFile( obj ); 
      }
    };

### 六，传递对象参数代理过长的参数列表

    var setUserInfo = function( id, name, address, sex, mobile, qq ){ 
      console.log( 'id= ' + id );
      console.log( 'name= ' +name );
      console.log( 'address= ' + address );
      console.log( 'sex= ' + sex ); console.log( 'mobile= ' + mobile ); console.log( 'qq= ' + qq );
    };
    setUserInfo( 1314, 'sven', 'shenzhen', 'male', '137********', 377876679 );

    // 把参数用一个对象包裹，不用再关系参数的数量和顺序
    var setUserInfo = function( obj ){
      console.log( 'id= ' + obj.id );
      console.log( 'name= ' + obj.name );
      console.log( 'address= ' + obj.address );
      console.log( 'sex= ' + obj.sex );
      console.log( 'mobile= ' + obj.mobile );
       console.log( 'qq= ' + obj.qq );
    };

    setUserInfo({
      id: 1314,
      name: 'sven',
      address: 'shenzhen',
      sex: 'male',
      mobile: '137********',
      qq: 377876679
    });

### 七，分解大型类

    var Spirit = function( name ){
      this.name = name;
    };
    Spirit.prototype.attack = function( type ){
      if ( type === 'waveBoxing' ){
        console.log( this.name + ': 使用波动拳' );
      }else if( type === 'whirlKick' ){
        console.log( this.name + ': 使用旋风腿')
        }
    };

    var spirit = new Spirit( 'RYU' );
    spirit.attack( 'waveBoxing' );
    spirit.attack( 'whirlKick' );


    // Spirit.prototype.attack 如果要添加方法，那会变得很大，所以必要作为一个单独的类存在
    var Attack = function( spirit ){
      this.spirit = spirit;
    };

    Attack.prototype.start = function( type ){
      return this.list[ type ].call( this );
    };

    Attack.prototype.list = {
      waveBoxing: function(){
        console.log( this.spirit.name + ': 使用波动拳' );
      },
      whirlKick: function(){
        console.log( this.spirit.name + ': 使用旋风腿' );
      }
    };

