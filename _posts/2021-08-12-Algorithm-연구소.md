---
title:  "백준 14502 연구소"
excerpt: "알고리즘 테스트"
categories:
  - Algorithm
tag: 
  - Algorithm
toc: true  
---

## [백준 연구소 ](https://www.acmicpc.net/problem/14502 "백준 연구소")

DFS를 통해 모든 빈곳의 벽을 만들고 BFS를 통해 좀비를 감염시켜 정상인이 많은 경우를 확인하여 구현했다.

막상 구현할때는 크게 어렵지 않았는데 처음에 어떻게 구현해야 할지 떠오르지가 않아 구글 검색으로 DFS+BFS 조합을 알아냈다.

처음엔 BFS 탐색시 메소드 4개를 빈곳이면 채워나가려고 했는데 처음 위치는 좀비였는데 빈곳일때만 재귀 호출을 가능하게 만들어 실패... 

배열을 통해 미리 위치를 지정하고 좀 for문을 통해 구현해서 통과했다
 
- 소스 코드   
 
``` java
import java.util.Scanner;

public class Laboratory {

	static int limit = 3;
	static int n, m;
	static int[][] map;
	static int result = 0;
	static int[][] cloneMap;
	static int[] nx = { 1, -1, 0, 0 };
	static int[] ny = { 0, 0, 1, -1 };

	public static void main(String[] args) {
		Scanner sc = new Scanner(System.in);

		n = sc.nextInt();
		m = sc.nextInt();
		map = new int[n][m];
		cloneMap = new int[n][m];

		//맵 셋팅
		for (int i = 0; i < n; i++) {
			for (int j = 0; j < m; j++) {
				map[i][j] = sc.nextInt();
			}
		}

		per(0);

		System.out.println(result);

	}

	private static void per(int cnt) {
		if (cnt == limit) {
			checkMap();
			return;
		}

		for (int i = 0; i < n; i++) {
			for (int j = 0; j < m; j++) {
				if (map[i][j] == 0) {
					map[i][j] = 1;
					per(cnt + 1);
					map[i][j] = 0;
				}

			}
		}

	}

	private static void checkMap() {
		// 맵 복사
		for (int i = 0; i < n; i++) {
			for (int j = 0; j < m; j++) {
				cloneMap[i][j] = map[i][j];

			}
		}
		// 좀비 감염
		for (int i = 0; i < n; i++) {
			for (int j = 0; j < m; j++) {
				if (cloneMap[i][j] == 2) {
					setPoison(i, j);
				}
			}
		}
		int normal = 0;
		// 생존자 확인
		for (int i = 0; i < n; i++) {
			for (int j = 0; j < m; j++) {
				if (cloneMap[i][j] == 0) {
					normal++;
				}
			}
		}

		result = Math.max(result, normal);

	}

	private static void setPoison(int i, int j) {

		for (int k = 0; k < 4; k++) {
			int x = i + nx[k];
			int y = j + ny[k];
			if (x >= 0 && y >= 00 && x < n && y < m) {
				if (cloneMap[x][y] == 0) {
					cloneMap[x][y] = 2;
					setPoison(x, y);
				}
			}

		}
	}
}

```