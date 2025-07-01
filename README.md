---
layout:
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# 👋 Yettie Vest 파라미터 난독화 가이드

> Author: kwook\
> Date: 2024.05.13 문서 생성\
> Date: 2025.04.09 Fetch API 호출 시 난독화 처리 추가로 인한 문서 업데이트

## 개요 <a href="#undefined" id="undefined"></a>

Yettie Soft의 VestWeb을 활용하여 파라미터 난독화

## 적용가이드 <a href="#undefined" id="undefined"></a>

아래 순서대로 진행해야 함.

### 1. VestWeb 서버 설정 <a href="#id-1-vestweb" id="id-1-vestweb"></a>

※ 여기서 서버는 개발/운영 뿐만 아니라 로컬이 될 수도 있다.

1. vestweb\_ver3.1.3.zip 파일을 확인한다.\
   HR5.0 SVN 소스를 받은 경우, /etc/ 폴더 내의 vestweb\_ver3.1.3.zip 가 있는지 확인하고,\
   그 외의 경우 [VestWeb\[링크\]](https://isugrp.sharepoint.com/:u:/s/ISUTEAMS_ORG_ISU_STISU_SY066-33/EcdN9ZfIMW1Iu0IhTJiBZ2sBWjJchqdNR5zE_L5oZPAqQw?e=XpJB7f)을 다운로드 받는다.
2. 서버에 압축 해제. 압축해제 시 아래와 같은 파일구조.\
   ![](http://localhost:63342/markdownPreview/641738029/etc/doc/VestWeb/image/vest01.png?_ijt=6772onq9cqvh6uions14mf4ujh)
3. /conf/vestweb\_conf.xml 파일 오픈. **(HR5.0의 경우 /WEB-INF/conf/vestweb\_conf.xml 파일 오픈)**
   * General 태그 내 'logDir', 'auditLogDir', 'securiptPath', 'pregenPath', 'submitPath'의 경로를 압축해제 한 VestWeb 폴더 경로로 수정. Windows 서버 환경일 경우 경로 구분자를 \\\\(역슬래시 2개)로 사용한다.\
     ![](http://localhost:63342/markdownPreview/641738029/etc/doc/VestWeb/image/vest02.png?_ijt=6772onq9cqvh6uions14mf4ujh)
   * Obfuscate 태그 내 'autoDecrypt' 를 **2** 로 수정.\
     ![](http://localhost:63342/markdownPreview/641738029/etc/doc/VestWeb/image/vest03.png?_ijt=6772onq9cqvh6uions14mf4ujh)
   * ExceptionPattern 태그 속성에 추가. (SSE 알림 예외 처리) ![](http://localhost:63342/markdownPreview/641738029/etc/doc/VestWeb/image/vest12.png?_ijt=6772onq9cqvh6uions14mf4ujh)
4.  서버OS가 윈도우일 경우 /bin/gen\_secuript.bat, 리눅스 계열일 경우 gen\_secuript.sh 파일 수정.

    * vestweb\*.jar, javarose\_application-1.6.2.jar, javarose\_crypto-1.6.2.jar 를 버전에 맞게 수정.\
      vestweb 3.1.3 버전 기준 lib 폴더 내 javarose\_application, javarose\_crypto 버전이 1.7.5 이므로,\
      **vestweb-3.1.3.jar, javarose\_application-1.7.5.jar, javarose\_crypto-1.7.5.jar** 로 수정.
    * CONFIG 경로를 vestweb\_conf.xml 파일 경로로 수정.\
      3번 에서 HR5.0의 /WEB-INF/conf/vestweb\_conf.xml 파일을 수정한 경우, 프로젝트경로 내 vestweb\_conf.xml 의 경로로 변경.

    \<gen\_secuript.bat>\
    ![](http://localhost:63342/markdownPreview/641738029/etc/doc/VestWeb/image/vest04.png?_ijt=6772onq9cqvh6uions14mf4ujh)\
    \<gen\_secuript.sh>\
    ![](http://localhost:63342/markdownPreview/641738029/etc/doc/VestWeb/image/vest05.png?_ijt=6772onq9cqvh6uions14mf4ujh)
5. 서버에 맞게 /bin/gen\_secuript 실행. 실행이 정상적으로 완료되면 /pregen/ 폴더에 새로운 폴더가 생성되어 있다.

## 2. 프로젝트 수정 <a href="#id-2" id="id-2"></a>

> ```
> HR5.0 SVN 체크아웃을 받은 경우, 2. 프로젝트 수정 절차는 무시해도 됩니다. (이미 수정하여 커밋됨) 
> ```

### 2-1. jar 파일 추가 <a href="#id-2-1-jar" id="id-2-1-jar"></a>

1. 위 VestWeb 폴더의 /lib/ 내 jar 파일 5개를 프로젝트의 lib 폴더로 이동.
2. 프로젝트 빌드 라이브러리에 추가. (이미 /lib/ 폴더를 프로젝트 빌드 라이브러리로 추가했다면 무관.)

### 2-2. Filter 추가 <a href="#id-2-2-filter" id="id-2-2-filter"></a>

> ```
> HR5.0 에는 WebMvcConfig.java 에 2-2 소스가 포함되어 있음.    
> ```

Filter 를 추가하기 위해 FilterConfig.java 파일을 생성하고,\
VestWeb 난독화된 파라미터를 자동 복호화하기 위해 VestWebFilter 를 Filter 에 등록.\
&#xNAN;**!!꼭 아래 config의 변수를 서버의 VestWeb 경로로 수정필요!!**\
**registrationBean.addInitParameter("config", "{서버 내 VestWeb경로}\\\conf\\\vestweb\_conf.xml");**

```java
package com.isu.cloudhr.common.config;

import com.yettiesoft.vestweb.operation.VestWebFilter;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.servlet.Filter;

@Configuration
public class FilterConfig {

    @Bean
    public FilterRegistrationBean<Filter> filterRegistrationBean() {
        FilterRegistrationBean<Filter> registrationBean = new FilterRegistrationBean<>();

        registrationBean.setFilter(new VestWebFilter());
        registrationBean.addUrlPatterns("/*");
        registrationBean.addInitParameter("config", "{서버 내 VestWeb경로}\\conf\\vestweb_conf.xml");
        registrationBean.setName("VestWebFilter");
        registrationBean.setOrder(Ordered.HIGHEST_PRECEDENCE);
        return registrationBean;
    }
}
```

### 2-3. Controller 추가 <a href="#id-2-3-controller" id="id-2-3-controller"></a>

Yettie Soft 의 난독화 관련 소스를 동적으로 불러오기 위한 VestWebController.java 추가.

```java
package com.isu.cloudhr.v1.controller;

import com.yettiesoft.vestweb.capsule.VestSubmit;
import com.yettiesoft.vestweb.exception.VestWebException;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.PrintWriter;

@Controller
public class VestWebController {

    @RequestMapping("/vestsubmit")
    public void getVestsubmit(HttpServletRequest request, HttpServletResponse response) throws Exception {
        response.setContentType("text/html");
        response.setCharacterEncoding("UTF-8");
        PrintWriter out = response.getWriter();
        try {
            VestSubmit.writeVestSubmit(/*request, */out);
        } catch (VestWebException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        out.close();
    }
}
```

### 2-4. VestSubmit.js 추가 <a href="#id-2-4-vestsubmitjs" id="id-2-4-vestsubmitjs"></a>

> ```
> HR5.0 에서는 /resources/static/common/js/VestSubmit.js 에 위치    
> ```

VestSubmit 소스를 가져오기 위한 VestSubmit.js 파일을 resources/static 경로에 추가.

```javascript
// VestSubmit.js
document.write("<script src='/vestsubmit'></script>");
```

### 2-5. VestUtils.js 추가 <a href="#id-2-5-vestutilsjs" id="id-2-5-vestutilsjs"></a>

> ```
> HR5.0 에서는 /resources/static/common/js/VestUtils.js 에 위치
> ```

모든 서버와의 통신할 때 HTML5 표준인 XMLHttpRequest를 Override 하여 전처리로 난독화 작업을 처리.

```javascript
/**
 * Vest Utils
 *
 * <pre>
 * YettieSoft Vest 난독화 관련 유틸
 *
 * Vest 난독화 유틸을 이용하여 ajax 통신을 위한 공통 처리 작업
 * HTML5 표준 XMLHttpRequest 를 override 하여 모든 통신(ajax, IBSheet doSearch 등...) 시 전처리로 Vest 난독화 작업을 진행 처리해준다.
 *
 * [2024.06.12 Update]
 *  1. 서버 페이징을 사용할 이유가 없어 JSP -> 정적 javascript 로 변경.
 *  2. 중복 로드로 인한 스크립트 오류 방지 코드 추가.
 *
 * [2025.04.09 Update]
 *  1. Fetch API 사용 시에도 난독화 처리될 수 있도록 소스 추가.
 * </pre>
 */

var VestUtils = {

        // Vest 난독화 여부
        ENC_YN: true,

        /**
         * 파라미터 난독화
         *
         * @param param 난독화할 파라미터
         * @returns 난독화된 파라미터
         */
        encParam: function(param) {
            //console.log("not encrypted params", param);
            if (this.isEmptyObj(param)) return "";

            if (this.isJsonObject(param)) {
                if (typeof VestAjaxJson !== 'function') return param; // 난독화 관련 function 존재 여부

                return JSON.stringify(VestAjaxJson(param));
            } else {
                if (typeof VestAjaxExt !== 'function') return param; // 난독화 관련 function 존재 여부

                return VestAjaxExt(this.getRemovedFirstAtParamString(param));
            }
        },

        /**
         * JSON 객체 여부 취득
         *
         * @param obj JSON 객체 여부롤 판단할 객체
         * @returns boolean JSON 객체 여부
         */
        isJsonObject: function(obj) {
            try {
                let json;
                if (typeof obj === 'object') {
                    json = JSON.parse(JSON.stringify(obj));
                } else {
                    json = JSON.parse(obj);
                }

                return (typeof json === 'object');
            } catch (e) {
                return false;
            }
        },

        /**
         * 빈 객체 여부 취득
         *
         * @param obj 빈 객체 여부롤 판단할 객체
         * @returns boolean 빈 객체 여부
         */
        isEmptyObj: function(obj) {
            return obj === undefined || obj === null || obj === '';
        },

        /**
         * 난독화 여부 조회.
         * 난독화 제외 case
         * 1. 파라미터 중 vestEncYn 값이 N일 경우
         * 2. 파일첨부일 경우
         * 3. 파라미터에 NOT_INCLUDE_KEYS 에 포함된 key 값을 가진 경우
         *
         * @param argStr 파라미터 String
         * @return boolean 난독화여부
         */
        isEnc: function(argStr) {
            const NOT_INCLUDE_KEYS = ["mrd_path", "rk"]; // 파라미터에 해당 키를 가지고 있는 경우 난독화하지 않는다. (RD 출력물 등을 위해)

            if (argStr instanceof FormData) return false; // 파일첨부의 경우 난독화 실행하지 않음.
            if (typeof argStr !== "string") return this.ENC_YN;

            if (this.isJsonObject(argStr)) {
                // JSON Object 형태의 파라미터 String 인 경우
                const jsonObj = JSON.parse(argStr);
                const isEncByChk = (Object.keys(jsonObj).filter(key => NOT_INCLUDE_KEYS.includes(key) || (key === "vestEncYn" && jsonObj["vestEncYn"] === "N")).length === 0);
                return isEncByChk && this.ENC_YN;
            } else {
                // 일반 파라미터 String 인 경우
                const isEncByChk = (argStr.split("&").filter(str => NOT_INCLUDE_KEYS.includes(str.split("=")[0]) || (str === "vestEncYn=N")).length === 0);
                return isEncByChk && this.ENC_YN;
            }
        },

        /**
         * JSON 을 QueryString 으로 변환
         *
         * @param objJson QueryString 으로 변환할 JSON 객체
         * @returns String QueryString 으로 변환된 문자열
         */
        convertJsonToQueryString: function(objJson) {
            if (this.isEmptyObj(objJson)) return "";

            try {
                return Object.entries(objJson).map(([key, value]) => (value && encodeURIComponent(key) + '=' + encodeURIComponent(value))).filter(v => v).join('&');
            } catch (e) {
                return "";
            }
        },

        /**
         * String 형태의 파라미터 제일 앞에 & 이 있는 경우 삭제하여 반환. 복호화 시 가장 앞에 & 표기가 있을 경우 에러를 발생함.
         * @param paramStr 파라미터 String
         * @returns {*|string} 반환된 값
         */
        getRemovedFirstAtParamString: function(paramStr) {
            if (typeof paramStr !== 'string') return paramStr;

            if (paramStr.indexOf("&") === 0) return paramStr.substring(1);
            else return paramStr;
        }
    };


(function(open, send) {
    // 중복 로드로 인한 오류 방지
    if (!window.isLoadedVestUtils) window.isLoadedVestUtils = true;
    else return;


    // HTML5 표준 XMLHttpRequest open, send function 의 prototype 을 재정의.
    // GET 메소드의 경우 open 시에 파라미터를 URL 과 같이 전달하기 때문에 open 에서 난독화를 진행한다.
    XMLHttpRequest.prototype.open = function(method, url, async) {
        // method 가 GET 인 경우 파라미터가 url 에 붙어 오기 때문에 open 시에도 url 난독화를 진행한다.
        if (method.toUpperCase() === "GET" && arguments[1]) {
            const urlArray = arguments[1].split("?");
            if (urlArray[1] && VestUtils.isEnc(urlArray[1])) {
                arguments[1] = urlArray[0] + "?" + VestUtils.encParam(urlArray[1]);
            }
        }

        open.apply(this, arguments);
    }

    // POST 메소드의 경우 send 시에 파라미터 데이터를 body 로 전달하기 때문에 send 에서 난독화 처리를 해야한다.
    XMLHttpRequest.prototype.send = function(body) {
        const callback = this.onreadystatechange;
        this.onreadystatechange = function() {
            if (callback) {
                callback.apply(this, arguments);
            }
        }

        const onload = this.onload;
        this.onload = function() {
            if (onload) {
                onload.apply(this, arguments);
            }
        }

        const onloadstart = this.onloadstart;
        this.onloadstart = function() {
            if (onloadstart) {
                onloadstart.apply(this, arguments);
            }
        }

        try {
            if (VestUtils.isEnc(arguments[0])) {
                // 난독화
                arguments[0] = VestUtils.encParam(arguments[0]);
            }
        } catch(e) {
            console.error(e);
        }

        send.apply(this, arguments);
    }
}(XMLHttpRequest.prototype.open, XMLHttpRequest.prototype.send));

/**
 * Fetch API 에서 VestWeb 난독화 솔루션을 적용하기 위한 재정의
 */
var {fetch: origFetch} = window;
window.fetch = async (...args) => {
    const isGETMethod = (args[1].method === "GET");
    if (isGETMethod) {
        // method 가 GET 인 경우 파라미터가 url 에 붙어 오기 때문에 url에서 난독화를 진행한다.
        const urlArray = args[0].split("?");
        if (urlArray[1] && VestUtils.isEnc(urlArray[1])) {
            args[0] = urlArray[0] + "?" + VestUtils.encParam(urlArray[1]);
        }
    } else {
        const body = args[1].body;
        if (body && VestUtils.isEnc(body)) {
            // 난독화
            args[1].body = VestUtils.encParam(body);
        }
    }
    return await origFetch(...args);
}
```

### 2-6. 공통 script 에 Vest난독화 관련 소스 import <a href="#id-2-6-script-vest-import" id="id-2-6-script-vest-import"></a>

> ```
> HR5.0 에는 header.jsp, jqueryScript.jsp 에 아래 내용이 포함되어 있다.    
> ```

**VestSubmit.js, VestUtils.js 실제 파일 경로에 맞추어 작성한다.**

```javascript
<!-- Vest -->
<script src="/common/js/VestSubmit.js"></script>
<script src="/common/js/VestUtils.js"></script>
```

## 3. 원리 <a href="#id-3" id="id-3"></a>

> ```
> 모든 서버와의 호출은 HTML5 표준 XMLHttpRequest 를 따르기 때문에, XMLHttpRequest 의 open (서버 통신 설정), send (서버 통신) 의 프로토타입을 재설정하여 모든 서버와의 호출 (JQuery ajax, IBSheet doSearch, doSave 등) 시에 전처리되도록 한다.
> [2025.04.09] Fetch API 사용 시에도 난독화 처리되도록 추가함.
> ```

> ```
> VestSubmit.js 에서 /vestsubmit 을 호출하여 VestWeb 의 난독화 용도의 함수가 생성되면 VestAjax, VestAjaxExt, VestAjaxJson 등의 함수를 통해 난독화가 가능하다.    
> ```

> ```
> 난독화를 하면 파라미터 또는 Request Body 에 b, g 라는 데이터가 생성된다. 이 b, g 라는 파라미터가 존재하는 경우, VestWebFilter 에서 b, g 라는 파라미터가 정상적인 데이터로 복호화되어 Controller 의 파라미터로 넘어간다.    
> ```

## 4. 테스트 <a href="#id-4" id="id-4"></a>

### WAS 시작 시 <a href="#was" id="was"></a>

VestWeb 필터가 정상적으로 적용이 된 경우 아래 이미지와 같이 WAS 시작 시 콘솔 로그에 VestWeb realPath 가 표시됨. ![](http://localhost:63342/markdownPreview/641738029/etc/doc/VestWeb/image/vest07.png?_ijt=6772onq9cqvh6uions14mf4ujh)

### 브라우저 개발자 모드 <a href="#undefined" id="undefined"></a>

#### 통신유형이 GET 인 경우 <a href="#get" id="get"></a>

아래와 같이 URL 뒤에 g, b라는 난독화된 파라미터로 호출한다.\
![](http://localhost:63342/markdownPreview/641738029/etc/doc/VestWeb/image/vest08.png?_ijt=6772onq9cqvh6uions14mf4ujh)

#### 통신유형이 POST 인 경우 <a href="#post" id="post"></a>

**데이터 조회**

![](http://localhost:63342/markdownPreview/641738029/etc/doc/VestWeb/image/vest09.png?_ijt=6772onq9cqvh6uions14mf4ujh)

**데이터 저장**

![](http://localhost:63342/markdownPreview/641738029/etc/doc/VestWeb/image/vest11.png?_ijt=6772onq9cqvh6uions14mf4ujh)

#### Layer 팝업 호출 시 <a href="#layer" id="layer"></a>

![](http://localhost:63342/markdownPreview/641738029/etc/doc/VestWeb/image/vest10.png?_ijt=6772onq9cqvh6uions14mf4ujh)

## 5. 유의사항 <a href="#id-5" id="id-5"></a>

A. 파일첨부의 경우 난독화가 되지 않음. VestUtils.jsp 내 isEnc 함수에서 파일첨부 케이스인지 여부 체크.\
B. 파라미터에 vestEncYn 로 값이 'N' 이 들어올 경우, 난독화를 진행하지 않음.\
혹시 난독화가 필요없거나 사용하지 말아야 할 경우 vestEncYn 파라미터를 사용할 것.\
C. 파라미터에 g 또는 b 라는 키는 사용 불가. (VestWeb 의 난독화에 사용되기 때문)\
D. 출력물(RD)은 꼭 테스트가 필요함!\
E. /vestsubmit 호출 시 WAS 의 heap 메모리를 작게 설정할 경우 OutOfMemoryException 이 발생할 수 있음. 그 때는 메모리 사이즈를 조절할 것.
