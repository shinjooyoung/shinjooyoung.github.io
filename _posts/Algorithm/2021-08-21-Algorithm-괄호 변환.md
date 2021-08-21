---
title:  "프로그래머스 괄호 변환"
excerpt: "알고리즘 테스트"
categories:
  - Algorithm
tag: 
  - Algorithm
  - CodingTest
toc: true  
---

## [괄호 변환](https://programmers.co.kr/learn/courses/30/lessons/60058 "프로그래머스")

문제 설명을 읽어도 어떻게 하라는 건지 이해가 잘 가지 않아 계속 읽음..  
읽어도 잘 모르겠어서 설명에 나와 있는 그대로 구현해서 통과함


코드작성은 어렵지 않게 했는데 문제를 이해하는게 어려웠다.

 
``` java
public static class Solution {
	public static String solution(String p) {
		String answer = "";
		// 올바른 괄호 문자열 확인
		if (confirmStr(p))
			return p;
		answer = generateStr(p);
		return answer;
	}

	private static String generateStr(String str) {
		// 빈 문자열 반환

		if (str.equals("")) {
			return "";
		}

		String u = "";
		String v = "";
		int left = 0;
		int right = 0;
		for (int i = 0; i < str.length(); i++) {
			if (str.charAt(i) == '(') {
				left++;
			} else {
				right++;
			}
			u += Character.toString(str.charAt(i));
			if (left == right) {
				if (i < str.length() - 1) {
					v = str.substring(i + 1, str.length());
				}
				break;
			}
		}
		if (confirmStr(u)) {
			u += generateStr(v);
		} else {
			String emptyStr = "(";
			emptyStr += generateStr(v);
			emptyStr += ")";
			for (int i = 1; i < u.length() - 1; i++) {
				if (u.charAt(i) == '(') {
					emptyStr += ")";
				} else {
					emptyStr += "(";
				}
			}
			return emptyStr;
		}
		return u;
	}

	private static boolean confirmStr(String str) {
		boolean result = true;
		int left = 0;
		int right = 0;
		for (int i = 0; i < str.length(); i++) {
			if (str.charAt(i) == '(') {
				left++;
			} else {
				right++;
			}
			if (left < right) {
				result = false;
				break;
			}
		}
		return result;
	}
}
```