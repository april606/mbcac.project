# mbcac.project
## 팀 프로젝트 안내
* 주제: 잘 하자
* 기간: 3달
  +1달: 분석
  +1달: 설계
  +1달: 구현
   - 1주: 로그인
   - 2주: 게시판
   - 3주: 뉴스

## 아래 코드를 참고하세요
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8" trimDirectiveWhitespaces="true"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt" %>

<jsp:useBean id="bdao" class="com.mbcac.boardT2.BoardDAO" />
<jsp:useBean id="b" class="com.mbcac.boardT2.BoardVO" />
	<jsp:setProperty name="b" property="*" />
	
<c:set var="added" value="${bdao.addedBoard(b)}" />

{"added":${added}}
