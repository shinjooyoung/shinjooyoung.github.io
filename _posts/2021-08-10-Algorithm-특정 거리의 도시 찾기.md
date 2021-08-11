---
title:  "백준 18352 특정 거리의 도시 찾기"
excerpt: "알고리즘 테스트"
categories:
  - Algorithm
tag: 
  - Algorithm
toc: true  
---

## [백준 특정거리의 도시 찾기 ](https://www.acmicpc.net/problem/18352 "백준 특정거리의 도시 찾기")

- 도시 갯수만큼 클래스 리스트를 만들어 큐를 사용해 구현함
 
- 소스 코드   
 
``` java
  
public class Main {
	static class City{
		int next;
		int cost;
		
		City(int next, int cost) {
			this.next = next;
			this.cost = cost;
		}
	}
	
	static int n,m,k,x;
	static ArrayList<Integer> result = new ArrayList<Integer>();
	
	static boolean[] visit; 
	static ArrayList<ArrayList<City>> citys;
	
	public static void main(String[] args) {
		
		Scanner sc = new Scanner(System.in);
		
		//도시의 갯수
		n = sc.nextInt();
		//도로 갯수
		m = sc.nextInt();
		//거리 정보
		k = sc.nextInt();
		//출발 도시
		x = sc.nextInt();
		
		// 첫방문 체크
		visit = new boolean[n+1];
		
		citys = new ArrayList<ArrayList<City>>();
		
		// 도시만큼 arry 생성
		for(int i = 0; i <= n; i++) {
			citys.add(new ArrayList<City>());
		}
		
		// 도로에 해당하는 city 입력
		for(int i = 0; i < m; i++) {
			int a = sc.nextInt();
			int b = sc.nextInt();
			citys.get(a).add(new City(b, 1));
		}
		
		findCity();
		
		if(result.size() <= 0) {
			System.out.println(-1);
		} else {
			Collections.sort(result);
			for(int num : result) {
				System.out.println(num);				
			}
		}

	}
	private static void findCity() {
		// 큐를 이용해 연결된 도로를 확인 후 입력 최단거리를 찾아냄
		
		Queue<City> q = new LinkedList<City>();
		
		// 자신의 위치는 0
		q.offer(new City(x,0));
		visit[x] = true;
		while(!q.isEmpty()) {
			City now = q.poll();
			
			int nowCity = now.next;
			int nowCost = now.cost;
			
			ArrayList<City> nextCitys = citys.get(nowCity);
			
			for(int i = 0;i < nextCitys.size(); i++) {
				City nextCity = nextCitys.get(i);
				
				//한번 방문한 경우 제외
				if(visit[nextCity.next]) {
					continue;
				}
				// 방문처리
				visit[nextCity.next] = true;
				//거리확인 
				if(nextCity.cost + nowCost < k) {
					// 최단거리보다 작을경우 cost 갱신 후 다음 큐에 입력
					nextCity.cost = nextCity.cost + nowCost;
					q.offer(nextCity);
					continue;
				}
				if(nextCity.cost + nowCost == k) {
					result.add(nextCity.next);
				}
			}
		}
	}
}

```