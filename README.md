# mbcac.project
## Windows에서 자격증명 확인 / github 관련 자격증명 삭제

## github.com 회원가입

## 사이트에서 Repository 생성
  * Repository 주소 복사

## git 다운로드/설치
  * user.name / use.email 등록
    + git config --global user.name = "개인아이디"
    + git config --global user.
  * git config --list : 등록된 항목 확인

## github 토큰 신청/발급
  * github.com 화면 우측 상단 아이콘 클릭 > Settings > 왼쪽 메뉴 하단 Developer settings > Personal access tokens >
  * Tokens(classic) > Generate new token > Generate new token(classic) >
  * Note:간단 설명 입력 > Generate token

## 로컬 Repository에 원격 리파지토리 복사
  * 원격 저장소에 등록된 프로젝트를 최초로 로컬 저장소에 복제한다
  * git init
  * git clone [원격 저장소 주소]
  * git config --list  <-- origin 이라는 이름으로 저장소의 주소가 등록된 것을 확인


## 팀 프로젝트 안내
* 주제: 열심히 하자
* 기간: 3달
  + 1달: 분석
  + 1달: 설계
  + 1달: 구현
    - 1주: 로그인
    - 2주: 게시판
    - 3주: 뉴스

<a name="top"></a>

* [go to code1](#code1)
* [go to code2](#code2)
* [go to code3](#code3)
* [go to code4](#code4)
* [go to code5](#code5)


## 아래 코드를 참고하세요
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8" trimDirectiveWhitespaces="true"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt" %>

<jsp:useBean id="bdao" class="com.mbcac.boardT2.BoardDAO" />
<jsp:useBean id="b" class="com.mbcac.boardT2.BoardVO" />
	<jsp:setProperty name="b" property="*" />
	
<c:set var="added" value="${bdao.addedBoard(b)}" />

{"added":${added}}
```
[github마크다운으로 색상 설정하기](https://gist.github.com/luigiMinardi/4574708d404cdf4fe0da7ac6fe2314db)

$\color{#ff0000}{\textsf{색상설정}}$


<a name="code1">code1</a> [go to top](#top)
	
```jsp
	package com.mbcac.boardT2;

import java.sql.CallableStatement;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Types;
import java.util.ArrayList;
import java.util.List;

public class BoardDAO <Board>
{
	private Connection conn;
	private PreparedStatement pstmt;
	private ResultSet rs;
	
	private Connection getConn()
	{
		try {
			Class.forName("oracle.jdbc.OracleDriver");
			conn = DriverManager.getConnection(				
	                  "jdbc:oracle:thin:@localhost:1521:xe", "SCOTT", "TIGER");
			return conn;
		} catch (Exception e) {
			e.printStackTrace();
		}			
        return null;
	}
```
<a name="code2">code2</a> [go to top](#top)
```jsp	
	public List<BoardVO> getHierarchicalList()
	{
		getConn();
		String sql = "SELECT bnum, LPAD('└Re:',(LEVEL-1)*3,'　')|| title AS title, author, rdate"
					+ "FROM board_t2"
					+ "START WITH parent=0"
					+ "CONNECT BY PRIOR bnum=parent";
		
		List<BoardVO> list = new ArrayList();
				
		try {
			pstmt = conn.prepareStatement(sql);
			rs = pstmt.executeQuery();
			int ttlPages = 0;
			
			while(rs.next())
			{
				int bnum = rs.getInt("BNUM");
				String title = rs.getString("TITLE");
				String author = rs.getString("AUTHOR");
				java.sql.Date rdate = rs.getDate("RDATE");
				
				BoardVO b = new BoardVO();
				b.setBnum(bnum);
				b.setTitle(title);
				b.setAuthor(author);
				b.setRdate(rdate);
				
				list.add(b);
			}
			return list;
			
		} catch (SQLException e) {
			e.printStackTrace();
		} finally {				
	    	closeAll();				
	    }
		return null;
	}
```
<a name="code3">code3</a> [go to top](#top)

```jsp
	public Pagination <BoardVO> getHierarchicalList(int page, int IPP)
	{
		System.out.println("요청페이지: "+page);
		
		getConn();
		String sql2 = "SELECT * FROM "
					+" ("
					+"	SELECT t2.*, FLOOR((RN-1)/"+ IPP +"+1) page FROM"
					+"  ("
					+"   SELECT t1.*, ROWNUM RN FROM"
					+"	 ("
					+"    SELECT * FROM"		
					+"    ("
					+"     SELECT bnum, LPAD('└Re: ', (LEVEL-1)*4, '　') || title AS title, author, rdate, parent, ttlpages FROM "
					+"     ("
					+"		SELECT * FROM board_t2 CROSS JOIN (SELECT CEIL(COUNT(*)/"+ IPP +") ttlpages FROM board_t2)"	
					+"		ORDER BY rdate DESC"
					+"	   )"
					+"	   START WITH parent=0 CONNECT BY PRIOR bnum=parent"
					+"	  )"
					+"   )t1"
					+"  )t2"
					+" )"
					+"WHERE page=?";
		
		List <BoardVO> list = new ArrayList<>();
		
		try {
		pstmt = conn.prepareStatement(sql2);
		pstmt.setInt(1, page);
		rs = pstmt.executeQuery();
		int ttlPages = 0;
		
		while(rs.next())
		{
			int bnum = rs.getInt("BNUM");
			String title = rs.getString("TITLE");
			String author = rs.getString("AUTHOR");
			java.sql.Date rdate = rs.getDate("RDATE");
			ttlPages = rs.getInt("TTLPAGES");
			
			BoardVO board = new BoardVO();
			board.setBnum(bnum);
			board.setTitle(title);
			board.setAuthor(author);
			board.setRdate(rdate);
			
			list.add(board);
		}
		
		Pagination <BoardVO> pagination = new Pagination(page, IPP, ttlPages);
		pagination.setItems(list);
		return pagination;
	} catch (SQLException e) {
		e.printStackTrace();
	}finally {				
	   	closeAll();				
	   }
	return null;
	}
```
<a name="code4">code4</a> [go to top](#top)
```jsp
	private void closeAll() 
	{
		try {
			if(rs!=null) rs.close();
			if(pstmt!=null) pstmt.close();
	        if(conn!=null) conn.close();
		} catch (SQLException e) {
			e.printStackTrace();
		}
	}
	
	public BoardVO bByBnum(int bnum)
	{
		getConn();
		String sql = "SELECT * FROM board_t2 WHERE bnum=?";
		
		try {
			pstmt = conn.prepareStatement(sql);
			pstmt.setInt(1, bnum);
			rs = pstmt.executeQuery();
			
			if(rs.next()) {
				int bn = rs.getInt("BNUM");
				String title = rs.getString("TITLE");
				String author = rs.getString("AUTHOR");
				java.sql.Date rdate = rs.getDate("RDATE");
				String contents = rs.getString("CONTENTS");
				
				BoardVO b = new BoardVO();
				b.setBnum(bn);
				b.setTitle(title);
				b.setAuthor(author);
				b.setRdate(rdate);
				b.setContents(contents);
				return b;
			}
			
		} catch (SQLException e) {
			e.printStackTrace();
		}finally {				
	    	closeAll();				
	    }
		return null;
		
	}
```
<a name="code5">code5</a> [go to top](#top)
```jsp
	public int addedBoard(BoardVO b)
	{
		getConn();
		String sql = "INSERT INTO BOARD_T2(bnum,title,author,contents,rdate,parent)"
				+ "VALUES(board_seq2.NEXTVAL,?,?,?,SYSDATE,?)"
				+ "RETURNING bnum INTO ?";
		
		try {
			CallableStatement cstmt = conn.prepareCall("{call "+ sql + "}");
			
			cstmt.setString(1, b.getTitle());
			cstmt.setString(2, b.getAuthor());
			cstmt.setString(3, b.getContents());
			cstmt.setInt(4, b.getParent());
			cstmt.registerOutParameter(5, Types.INTEGER);
			int rows = cstmt.executeUpdate();
			int newId = cstmt.getInt(5);
			
			return rows>0 ? newId : 0;
			
		} catch (SQLException e) {
			e.printStackTrace();
		} finally {				
	    	closeAll();				
	    }
		
		return -1;
				
	}
	
}
```
