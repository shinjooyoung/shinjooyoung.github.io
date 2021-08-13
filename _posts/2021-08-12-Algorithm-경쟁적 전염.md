---
title:  "백준 18405 "
excerpt: "알고리즘 테스트"
categories:
  - Algorithm
tag: 
  - Algorithm
toc: true  
---

## [경쟁적 전염 ](https://www.acmicpc.net/problem/18405 "백준 연구소")

바이러스가 저장된 곳을 ArrayList를 통해  입력해두고 하나씩 추가될때마다 add를 해준다음 BFS를 통해 검사를 진행했다.

바이러스를 작은것부터 하나씩 ArrayList size만큼 for문을 도는데 처음에 size값을 변수처리를 하지 않아 계속 size가 늘어나면서 처음 바이러스만 퍼지면서 실패했다

변수 처리 후 테스트 입력값 통과

백준에서의 채점은 내일..

 
- 소스 코드   
 
``` java
package com.project;

import java.util.Scanner;


public class main {

    public static class Virus{
        int x;
        int y;
        
        Virus(int x, int y){
          this.x = x;
          this.y = y;
          
        } 
    }
    
    static int n,k,s,x,y;
    static int[][] map;
    static int[] nx = {1,-1,0,0}
    static int[] ny = {1,-1,0,0}
    
    static ArrayList<ArrayList<Virus>> infos = new ArrayList<ArrayList<Virus>>();
    
    public static void main(args[]){
      Scanner sc = new Scanner(System.in);
      
      //시험관 크기
      n = sc.nextInt();
      
      //바이러스 종류
      k = sc.nextInt();
      
      map = new int[n+1][n+1];
      
      //바이러스 종류별 위치 셋팅
      for(int i = 0; i <=k; i++){
        infos.add(new ArrayList<Virus>());
      }
      
      //전체 셋팅
      for(int i = 1; i <=n; i++){
        for(int j = 1; j <=n; j++){
           int num = sc.nextInt();
           map[i][j] = num;
           infos.get(num).add(new Virus(i,j,num));
        }
      }      
      
      // 시간
      s = sc.nextInt();
      x = sc.nextInt();
      y = sc.nextInt();
      
      spreaVirus();
    }
    
    private static void spreaVirus(){
      for(int i = 1; i <=s; i++){
        for(int a = 1; a <=k; a++){
           ArrayList<Virus> virusArr = infos.get(a);
           
           int size = virusArr.size();
           
           for(int j = 0; j < size; j++){
             Virus virus = virusArr.get(j);
             
             setVirus(virus.x, virus.y, a);
           }
           
        }
      }   
      
    }
    
    private static setVirus(int vx, int vy, int cate){
      for(int i = 0; i < 4; i++){
        int vnx = vx + nx[i];
        int vny = vy + ny[i];
        
        if(vnx >0 && vny >0 && vnx <= n && vny <= n){
          if(map[vnx][vny] == 0){
            map[vnx][vny] = cate;
            infos.get(cate).add(new Virus(vnx, vny));
          }
        }
        
      }
      
    }
}
```