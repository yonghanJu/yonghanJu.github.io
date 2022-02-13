---
layout: post
title:  Resursive Tiling
date:   2021-06-04 15:01:35 +0300
categories:   Algorithm
---
# Recursive Matrix Multiplication (C언어)

<br>

### Recursive Matrix Multiplication 설명 
***

- Recursive Matrix Multiplication 기법은 정방행렬을 n/2×n/2 행렬  

  4개로 나눈 뒤 내적(inner product)해 공간복잡도를 줄이는 방법이다.

- 이같은 방식을 타일링(tiling)이라고 한다.

- A와 B를 내적해 C를 얻는 연산(C = AB)의 경우 다음과 같이   분할-정복 방식으로 나눠풀 수 있다.

<br>

**Recursive Matrix Multiplication**

<br>

- n×n 정방행렬을 내적하는 데 필요한 계산복잡도를 T(n)이라 할 때 이를 다음과 같이 분해할 수 있다.

 T(n) = T(divide) + T(conquer) + T(merge)

<br>

춡처: https://ratsgo.github.io/data%20structure&algorithm/2017/09/11/recurrence/ 

***

<br>

### 타일링 기법 사용시 공간 복잡도 이점
***
- 타일링 기법을 사용하지 않고 loop을 사용할 때

loop 을 통해 구현한 일반적인 함수의 간략 설명
```c
for(int a = 0; k < n; a++){
    for(int i = 0 ; i< n; i++){
        for(int j = 0 ; j< n; j++){
            sum += matrixA[k][j] * matrixB[j][i]
        }
    }
 }
 ```
함수가 호출되면 stack 메모리 위에 함수가 저장되고 함수가 끝나면 자동으로
stack 메모리 상에서 사라지게 된다.
위와 같은 코드로 함수를 만들면 모든 계산이 끝날 때까지 입력행렬의 모든 항을
읽고 stack 메모리에 쌓이게 된다.

<br>

- 타일링 기법 사용시

 Recursive Matrix Multiplication 의 구현 코드를 보면
각 항들을 바로 불러오는 게 아니라 n 의 값 즉 크기가 1 이 될 때까지 행렬을
잘게 나눈 다음 n==1 일 때 한 번만 계산하게 되며 자로 함수가 종료된다.

함수가 종료되면 stack 메모리에서 사라지게 되고 각각의 연산을 거치는 순간
바로 메모리에서 지워지게 된다.

따라서 Tiling 기법을 사용해 코드를 구현하면 모든 연산을 끝낼 때 까지 계속
메모리에 쌓이는 기존 방식과 다르게 연산 직후 메모리에서 사라져 높은
공간활용도를 갖을 수 있게 된다.

<br>

***

### 시간 복잡도 계산

***
**Recursive Matrix Multiplication 시간 복잡도**

- n = 1 일 때,    

        output   +=   matrixA * matrixB; 
        T(merge)         T(conquer)

두 값을 곱한 후 결과를 output 행렬에 더한다
T(1) = C(상수)

<br>

- n != 1 일 때,

 n 자리에 n/2 대입한 함수 8 번 재귀

<br>

```c 
T(n) = 8T(n/2)
= 8(8T(n/4))
= 8(8(8T(n/8)))
        …
= 8^log(2,n)* T(1)
= 2^3log(2,n) * C
= C * n^3
```

<br>

big-O 시간 복잡도는 **O(n^3)**로 시간복잡도 면에서 치명적이다.

<br>

```c
#include <stdio.h>
#include <stdlib.h>

/*  제목: Recursive Matrix Multiplication
    생성자:주용한     생성날짜:2021.03.21
    n * n 크기의 두 행렬을 n/2*n/2 크기로 나누어 재귀호출하고
    연산값을 output행렬에 저장한다 
*/

/*  함수 파라미터 순서대로 설명
    *a[], *b[]  => 입력받은 두 행렬
    *out[]      => 결과값을 담을 행렬
    n           => 행렬의 크기
    r1, c1, r2, c2  =>곱할 두 값의 좌표
    outR, outC  => 결과값이 입력될 좌표, 예를 들어 두 함수의
                 outR, outC값이 같으면 같은 곳에 저장.
*/  
void recurMultiply(int *a[], int *b[], int *out[], int n, int r1, int c1, int r2, int c2, int outR, int outC){

    if(n==1){
        //나누어진 타일을 연산 후 결과행렬에 더한다.
        *(out[outR]+outC) += (*(a[r1]+c1)) * (*(b[r2]+c2));
    }else{
        //두 행렬을 각각 4개로 나누고 재귀호출한다
        //C(11) = A(11)*B(11) + A(12)*B(21)      -outR, outC 공유   
        recurMultiply(a, b, out, n/2, r1+0,c1+0,r2+0,c2+0, outR+0,outC+0);     
        recurMultiply(a, b, out, n/2, r1+0,c1+n/2,r2+n/2,c2+0, outR+0,outC+0); 

        //C(12) = A(11)*B(12) + A(12)*B(22)      -outR, outC 공유
        recurMultiply(a, b, out, n/2, r1+0,c1+0,r2+0,c2+n/2, outR+0,outC+n/2);        
        recurMultiply(a, b, out, n/2, r1+0,c1+n/2,r2+n/2,c2+n/2, outR+0,outC+n/2);  

        //C(21) = A(21)*B(11) + A(22)*B(21)     -outR, outC 공유
        recurMultiply(a, b, out, n/2, r1+n/2,c1+0,r2+0,c2+0, outR+n/2,outC+0);
        recurMultiply(a, b, out, n/2, r1+n/2,c1+n/2,r2+n/2,c2+0, outR+n/2,outC+0);

        //C(22) = A(21)*B(12) + A(22)*B(22)     -outR, outC 공유
        recurMultiply(a, b, out, n/2, r1+n/2,c1+0,r2+0,c2+n/2, outR+n/2,outC+n/2);
        recurMultiply(a, b, out, n/2, r1+n/2,c1+n/2,r2+n/2,c2+n/2, outR+n/2,outC+n/2);
    }
}

int main(void){
    printf("크기 n과 두 행렬을 입력하시오.\n");
    // n = 행, 열의 크기
    int n; 
    scanf("%d", &n);

    //입력받을 두 행렬과 결과행렬 1개를 선언
    int* pmatrixA[n];
    int* pmatrixB[n];
    int* output[n];
    for(int i =0; i < n; i++){
        pmatrixA[i] = (int*)malloc(n*sizeof(int));
        pmatrixB[i] = (int*)malloc(n*sizeof(int));
        output[i] = (int*)malloc(n*sizeof(int));
    }

    //사용자로부터 입력행렬 받기
    for(int i=0; i<n; i++){
        for(int j=0; j<n; j++){
            scanf("%d",(pmatrixA[i]+j));
        }
    }
    for(int i=0; i<n; i++){
        for(int j=0; j<n; j++){
            scanf("%d",(pmatrixB[i]+j));
        }
    }

    //함수 실행
    recurMultiply(pmatrixA, pmatrixB, output, n, 0,0,0,0,0,0);

    //결과행렬 출력
    printf("\n실행 결과\n");
    for(int i=0; i<n; i++){
        for(int j=0; j<n; j++){
            printf("%d ",*(output[i]+j));
        }
        printf("\n");
    }

    for(int i =0; i < n; i++){
        free(pmatrixA[i]);
        free(pmatrixB[i]);
        free(output[i]);
    }
    return 0;
}
```
