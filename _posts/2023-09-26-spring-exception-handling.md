---
title: 자바 스프링에서 사용자 정의 예외처리하기
author: kjb4494
date: 2023-09-26 10:30:00 +0900
categories: [Backend, Java Spring]
tags: [java, spring, backend, controller advice, exception handling]
---

스프링 부트에서는 `@RestControllerAdvice`와 같은 예외처리를 손쉽게 관리할 수 있는 다양한 도구를 제공합니다. 이 글에서는 사용자 정의 예외처리와 공통 포멧으로 표준적인 HTTP 응답을 동시에 관리하는 방법을 살펴보겠습니다.

## 공통 응답 포멧을 위한 Dto

### ComResultDto

이 클래스는 기본적인 응답의 형태를 정의합니다. 응답에는 HTTP 상태 코드, 메시지, 그리고 사용자 정의 코드가 포함됩니다.

```java
@Getter @Setter @ToString
public class ComResultDto {
    private int httpStatusCode;
    private int code;
    private String message;

    public ComResultDto(ResponseCode rc) {
        this.httpStatusCode = rc.getHttpStatusCode();
        this.code = rc.getCode();
        this.message = rc.getMessage();
    }
}
```

### ComResponseDto

이 클래스는 응답 본문을 포함합니다. `ComResultDto`를 포함하여 응답 본문(`body`)도 포함할 수 있습니다.

```java
@Getter @Setter
public class ComResponseDto<T> {
    private ComResultDto result = new ComResultDto();
    private T body;
}
```

### ComResponseEntity

`ResponseEntity`를 확장하여 표준 HTTP 응답과 함께 사용자 정의 응답을 관리합니다.

```java
public class ComResponseEntity<T> extends ResponseEntity<ComResponseDto<T>> {
    public ComResponseEntity(ComResponseDto<T> body, HttpStatus status) {
        super(body, status);
        body.getResult().setHttpStatusCode(status.value());
    }
}
```

## 사용자 정의 예외 처리

### ResponseCode

각 예외 유형에 대한 사용자 정의 코드와 메시지, HTTP 상태 코드를 관리합니다. 이러한 정보는 이후 예외 처리에 사용됩니다.

```java
public enum ResponseCode {
    SYSTEM_001("com.fasterxml.jackson.core.io.JsonEOFException", 40000001, 400, "JSON 변환 중 문제가 있습니다."),
    SYSTEM_002("com.fasterxml.jackson.core.JsonParseException", 40000002, 400, "JSON 변환 중 문제가 있습니다."),
    SYSTEM_003("com.fasterxml.jackson.databind.exc.MismatchedInputException", 40000003, 400, "JSON 변환 중 문제가 있습니다."),
    // ...
    SYSTEM_028("org.springframework.dao.DuplicateKeyException", 50000031, 500, "중복된 값이 존재합니다."),
    SYSTEM_029("java.security.NoSuchAlgorithmException", 50000041, 500, "해시 변환 오류"),
    // CUSTOMS
    SUCCESS("success", 20000000, 200, "정상 처리 되었습니다."),
    INVALID("invalid", 40010001, 400, "입력값 오류 입니다."),
    ALREADY_EXISTS("already_exists", 40010002, 400, "이미 존재하는 데이터 입니다."),
    NOT_EXISTS("not_exists", 40010003, 400, "존재하지않는 데이터 입니다."),
    UNAUTHORIZED("unauthorized", 40110021, 401, "로그인이 필요합니다."),
    EXPIRED_JWT("expired_jwt", 40110022, 401, "만료된 토큰입니다."),
    INVALID_JWT("invalid_jwt", 40110023, 401, "유효하지않은 토큰입니다."),
    INVALID_REFRESH_TOKEN("invalid_refresh_token", 40110024, 401, "유효하지않은 토큰입니다."),
    EXPIRED_REFRESH_TOKEN("expired_refresh_token", 40110025, 401, "만료된 토큰입니다."),
    DISABLED_ACCOUNT("disabled_account", 40110026, 401, "사용 불가능한 계정입니다."),
    FORBIDDEN("forbidden", 40310001, 403, "이 페이지를 볼 수 있는 권한이 없습니다."),
    NOT_DEFINED("not_define", 50019999, 500, "정의되지않은 오류입니다."),
    ;

    // 키, 코드 등록
    private static final Map<String, ResponseCode> BY_KEY = new HashMap<>();
    private static final Map<Integer, ResponseCode> BY_CODE = new HashMap<>();
    static {
        for (ResponseCode rc: values()) {
            BY_KEY.put(rc.key, rc);
            BY_CODE.put(rc.code, rc);
        }
    }

    /**
     * 미들웨어 속성 키
     */
    public static final String MIDDLEWARE_KEY = "codeKey";

    private final String key;
    private final int code;
    private final int httpStatusCode;
    private final String message;

    ResponseCode(String key, int code, int httpStatusCode, String message) {
        this.key = key;
        this.code = code;
        this.httpStatusCode = httpStatusCode;
        this.message = message;
    }

    public static ResponseCode findByKey(String key) {
        return BY_KEY.getOrDefault(key, NOT_DEFINED);
    }

    public static ResponseCode findByCode(int code) {
        return BY_CODE.getOrDefault(code, NOT_DEFINED);
    }

    public static ResponseCode getSuccess() {
        return SUCCESS;
    }
}
```

### RestException

이 클래스는 비지니스 로직에서 예외처리를 발생시켜 RestControllerAdvice로 ResponseCode를 전달해주는 역할을 합니다.

```java
@Getter @Setter
public class RestException extends RuntimeException {
    private ResponseCode responseCode;
    private Throwable throwable = null;

    public RestException(ResponseCode responseCode) {
        this.responseCode = responseCode;
    }
}
```

### ComExceptionHandler

`@RestControllerAdvice` 어노테이션을 사용하여 예외를 처리합니다. `@RestControllerAdvice`는 컨트롤러에서 발생하는 예외를 처리하는 데 사용되는 어노테이션입니다. 이 어노테이션을 클래스에 추가하면 그 클래스는 전역 예외 핸들러 역할을 합니다. 이를 통해 컨트롤러에서 발생하는 예외를 한 곳에서 관리하고 처리할 수 있습니다.

```java
@Slf4j
@RestControllerAdvice
@RequiredArgsConstructor
public class ComExceptionHandler {
    /**
     * &#064;Valid  어노테이션 관련 처리
     * @return : ComResponseEntity<Map<String, Object>>
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public @ResponseBody ComResponseEntity<Map<String, Object>> handleMethodArgumentNotValidException(MethodArgumentNotValidException exception) {
        ComResultDto resultDto = new ComResultDto(ResponseCode.findByKey(exception.getClass().getName()));
        ComResponseDto<Map<String, Object>> result = new ComResponseDto<>();
        result.setResult(resultDto);

        // 1. body에 @Valid에서 발생한 상세 오류 정보 추가
        HashMap<String, Object> body = new HashMap<>();
        for (FieldError fieldError : exception.getBindingResult().getFieldErrors()) {
            body.put(fieldError.getField(), fieldError.getDefaultMessage());
        }
        result.setBody(body);

        ExceptionUtil.handlerCommonLogging(resultDto, exception);

        return new ComResponseEntity<>(result, HttpStatus.valueOf(resultDto.getHttpStatusCode()));
    }

    /**
     * 어플리케이션에서 정의하는 비지니스 오류처리
     * @return : ComResponseEntity<Void>
     */
    @ExceptionHandler(RestException.class)
    public @ResponseBody ComResponseEntity<Void> handleBizException(RestException exception) {
        ComResultDto resultDto = new ComResultDto(exception.getResponseCode());
        ComResponseDto<Void> result = new ComResponseDto<>();
        result.setResult(resultDto);

        ExceptionUtil.handlerCommonLogging(resultDto, exception);

        return new ComResponseEntity<>(result, HttpStatus.valueOf(resultDto.getHttpStatusCode()));
    }

    /**
     * Exception 클래스명 기준으로 미리 정의한 메세지 및 코드를 리턴
     * @return : ComResponseEntity<Void>
     */
    @ExceptionHandler(Exception.class)
    public @ResponseBody ComResponseEntity<Void> handleNoHandlerFoundException(Exception exception) {
        ComResultDto resultDto = new ComResultDto(ResponseCode.findByKey(exception.getClass().getName()));
        ComResponseDto<Void> result = new ComResponseDto<>();
        result.setResult(resultDto);

        ExceptionUtil.handlerCommonLogging(resultDto, exception);

        return new ComResponseEntity<>(result, HttpStatus.valueOf(resultDto.getHttpStatusCode()));
    }
}
```

1. **handleMethodArgumentNotValidException**

   이 메서드는 `@Valid` 어노테이션을 사용하여 검증이 실패한 경우를 처리합니다. `@Valid` 어노테이션은 주로 요청 본문의 유효성 검사에 사용되며, 유효하지 않은 필드가 있을 경우 `MethodArgumentNotValidException`이 발생합니다.

   이 메서드에서는 발생한 예외 정보와 함께 사용자 정의 응답을 생성하여 클라이언트에 반환합니다. 각 필드의 오류 메시지를 본문에 포함하여 클라이언트에게 어떤 필드가 유효하지 않은지 알립니다.

2. **handleBizException**

   이 메서드는 비즈니스 로직에서 발생하는 예외를 처리합니다. 여기에서 사용하는 `RestException`은 사용자가 정의한 예외 클래스로, 비즈니스 로직에서 발생할 수 있는 예외 상황을 나타냅니다.

   `RestException`에서 발생한 예외 정보를 바탕으로 사용자 정의 응답을 생성하고 반환합니다.

3. **handleNoHandlerFoundException**

   이 메서드는 다른 예외 핸들러에서 처리되지 않은 예외를 처리합니다. 여기에서는 예외 유형을 기반으로 `ResponseCode`를 찾아 사용자 정의 응답을 생성하고 반환합니다.

## 정리

이 구조를 사용하면 스프링 부트에서 발생하는 다양한 유형의 예외를 효과적으로 관리하고 처리할 수 있습니다. `@RestControllerAdvice`와 `@ExceptionHandler`를 사용하여 예외를 깔끔하게 처리할 수 있으며, 클라이언트에게 유용한 오류 메시지와 정보를 제공할 수 있습니다.
