# MicroProcessor Term project (Digital Clock)
# 마이크로프로세서 텀프로젝트 : 디지털 시계 만들기

## 목적 

그동안 쌓아온 프로그래밍 경험과 수업시간에 배운 하드웨어와 어셈블리어에 대한 지식을 활용하여 스스로 HW/SW를 제작하는 것이 목적이다. 구체적으론 소자 납땜을 완료하고 마이크로프로세서에 프로그램을 하여 시간과 분 그리고 초를 나타내는 디지털 시계를 제작하는 것이다.

마이크로프로세서를 활용하여 시간과 분 그리고 초를 나타내는 디지털 시계 제작하기

## 시스템 동작내용

![](./img/1.mp4)
![](./img/2.mp4)

1)하드웨어 추가 내용
### PORT A :
- (RA0~RA2) 스위치 1,2,3번 연결
- 스위치의 신호 1을 안정적으로 주기 위해 풀업 저항 추가
### PORT B : 
- (RB0~RB3) SEGMENT DIGIT 선택 단자 연결,
- (RB4) LED 연결
### PORT C :
- (RC0~RC7) SEGMENT A~DP 연결

2) 소프트웨어 추가 내용
###  기본 모드
-	시간과 분을 표시하는 모드
-	처음 시작시 00시 00분으로 시작
-	DIGIT 1,2 번에 시간 표시
-	DIGIT 3,4 번에 분 표시
###  초 표시 모드
-	SW1 을 한번 누를 때마다 기본모드 <-> 초 표시모드 전환
-	DIGIT 1,2 번은 시/분 모드와 구별하기 위해 OFF시킨다.
-	DIGIT 3,4 번에 초 표시
###  시간 설정 모드
-	SW3을 한번 누르면 시간 설정 모드로 전환
-	이때 기본 모드와 구별하기 위해 모든 DIGIT의 DOT을 ON시킨다.
-	SW1을 누를 때마다 시간 변수 증가
-	SW2를 누를 때마다 분 변수 증가
-	SW3을 누르면 초 변수를 0으로 초기화 후 기본 모드로 전환
###  LED로 AM/PM 표시 기능 추가
###  1초에 한번씩 DIGIT 2의 DP 깜빡이는 기능 추가

## 기능 및 동작과정

1) 기본 모드 : 기본적으로 시간 생성 부분과 시간 표시 부분으로 나뉜다. 시간 생성 부분에서는 타이머 인터럽트를 이용한 1초 생성 코드를 기본으로 하는데 1초 10초 1분 10분 1시간 10시간 단위의 변수 값을 각각 증가시키고 초기화하는 과정을 통해 시간을 만들어낼 수 있다.
시간 표시 부분에서는 7 세그먼트의 4개 디지트가 모두 표시되어야 하기 때문에 한 디지트마다 필요한 시간변수를 불러와서 표시하는 동작을 빠르게 번갈아 가며 반복한다.

2) 초 표시 모드 : SW1 입력을 통해 기본모드와 변환할 수 있다. 이때는 디스플레이 모드 변수의 0번 비트를 1로 만들어 기본 모드와 구별할 수 있게 하였다. 인터럽트의 디스플레이 구문에서 디스플레이 모드 변수를 비교하여 1인 경우 기본 모드 대신 초 표시 모드의 내용(1초,10초 변수)을 불러와 나타내도록 하였다.

3) 시간 설정 모드 :  PORTA의 2비트, 즉 RB2에 1이 입력되었는 지 (SW3이 눌렸는 지 ) 를 확인한다. 눌렸으면 SWITCH_3 서브루틴을 CALL 하여 시간 설정 모드에 들어간다. 
SWITCH_3 서브 루틴은 먼저 스위치 채터링을 방지하기 위하여 DELAY 서브루틴을 두 번 부른다. 그리고 OPT_MODE를 증가시킨다. 시간 설정을 위하여 시간 변수인 D_10SEC, D_1SEC, D_MODE 를 전부 CLRF한다. 그리고 바로 OPT_TIME 서브루틴을 진행한다. OPT_TIME 서브루틴은 RA0, RA1, RA2를 무한으로 스캔한다. RA2, 즉 SW3가 눌린 것을 감지하면 기존 시간 표시 기능 모드로 돌아가기전 변수CLRF를 하는 OPT_END 서브루틴으로 간다. OPT_END 서브루틴에서는 채터링 방지용 DELAY 서브루틴 2번과 OPT_MODE, INT_CNT, IN_DNT 변수 3개를 전부 CLRF하고 M_LOOP로 돌아간다. 만약 SW3이 눌리기 전에 SW1, SW2가 눌린 것을 확인하면 각각에 맞추어 OPT_HOUR 또는 OPT_MINUTE 서브루틴을 CALL하게 된다. 각 서브루틴은 채터링 방지용의 DELAY 서브루틴을 CALL 하고 각각의 시간 변수 ( D_HOUR1 또는 D_MIN ) 를 INCF하게 된다. 자세한 변수 숫자의 설정은 기존 시간설정의 DISP와 비슷하므로 설명을 생략한다.

4) AM/PM 표시기능 : AM_PM변수를 두어 시간 생성 모드에서 시간 변수가 11시에서 12시로 넘어가는 경우에 시간을 0으로 초기화 시키는 부분에 AM_PM변수를 1 증가시키는 명령어를 추가하여 모든 디스플레이 루틴에서 AM_PM변수의 0비트를 검사해 1인경우 PORT B의 4번 비트에 1을 인가하고 0인경우는 0을 인가하도록 하였다.

5) DOT 깜빡임 기능 : 변수는 INT_DNT와 D_DOT을 사용한다. INT_DNT는 0.5초를 만드는 변수이고 D_DOT은 *****이다. INT_DNT는 INT_CNT의 딱 두 배만큼 인터럽트에서 증가하기 때문에 0.5초를 만들 수 있다. 0.5초가 만들어진 것을 확인하면 DOT_AD 서브루틴을 CALL 한다.
DOT_AD는 먼저 D_DOT변수를 증가시키고 INT_DNT를 초기화한다.  이후 DOT 깜빡임 기능을 수행할 DISP_HOUR1에서는 D_DOT의 변수의 0비트가 0인지 확인하고 맞다면 PORTC에 넣을 값의 BIT0을 IORLW B’00000001’을 사용하여 1로 만든다.

## 각 조원의 역할

### 박선우 :
 시간 생성 및 표시기능 제작, 초 표시 모드 제작, AM/PM 표시기능 제작, 기말 발표 
### 장진아 : 
시간 설정 프로그램 제작, DOT 깜빡임 기능 제작, 프로그램 주요 알고리즘 정리, 중간 발표
### 공동 수행 : 
하드웨어 제작 및 동작 확인, 종합 테스트 및 오류 수정, 보고서 작성 및 마무리

## 프로그램 전체 소스 코드

```
LIST P=16F84
 ; --- REGISTER FILES 선언 -------
 ; BANK 0
 INDF EQU 00H
 TMR0 EQU 01H
 PCL EQU 02H
 STATUS EQU 03H
 FSR EQU 04H
 PORTA EQU 05H
 PORTB EQU 06H
 PORTC EQU 07H
 EEDATA EQU 08H
 EEADR EQU 09H
 PCLATH EQU 0AH
 INTCON EQU 0BH
 ; BANK 1
 OPTION_REG EQU 81H
 TRISA EQU 85H
 TRISB EQU 86H
 TRISC EQU 87H
 EECON1 EQU 88H
 EECON2 EQU 89H
 ADCON1 EQU 9FH
; --- STATUS BITS 선언 -----
 IRP EQU 7
 RP1 EQU 6
 RP0 EQU 5
 NOT_TO EQU 4
 NOT_PD EQU 3
 ZF EQU 2 ; ZERO FLAG BIT
 DC EQU 1 ; DIGIT CARRY/BORROW BIT
 CF EQU 0 ; CARRY/BORROW FLAG BIT

;
DBUF1 EQU 23H
DBUF2 EQU 24H
W_TEMP EQU 25H		; 인터럽트 전 W레지스터 저장
STATUS_TEMP EQU 26H	; 인터럽트 전 상태 레지스터 저장
DISP_CNT EQU 27H 	; DISP_CNT변수 설정
INT_CNT EQU 28H		; 1초 만들기
D_1SEC EQU 29H		; 초 1의자리 표시
D_10SEC EQU 2AH		; 초 10의자리 표시
D_MIN1 EQU 2BH		; 분 1의자리
D_MIN10 EQU 2CH		; 분 10의자리
D_HOUR1 EQU 2DH		; 시간 1의자리
D_HOUR10 EQU 2EH	; 시간 10의자리
D_MODE EQU 2FH		; 디스플레이 모드 변수
D_DOT EQU 30H 		; 초 . 깜빡임
INT_DNT EQU 31H		; 0.5초 만들기
OPT_MODE EQU 32H	; 시간설정 모드 표시 변
AMPM EQU 33H		; AMPM변수
ORG 0
GOTO START_UP
ORG 4 ; 인터럽트서비스루틴(ISR) 번지
; ISR 시작 번지
MOVWF W_TEMP ; 현재 사용되고 있는 W REG를 저장
SWAPF	STATUS, W
MOVWF STATUS_TEMP ; 현재 STATUS REG를 저장
CALL DISP ; DISPLAY 부 프로그램
SWAPF STATUS_TEMP,W ; 저장된 내용으로 복원
MOVWF STATUS
SWAPF W_TEMP,F
SWAPF W_TEMP,W
BCF INTCON,2 ; TMR0의 overflow flag 비트를 0으로
RETFIE ; Return과 동시에 GIE enable 시켜야 인터럽트 다시 받음

; DISPLAY ROUTINE (Interrupt 걸리면 수행함)
DISP 
MOVLW .0
SUBWF DISP_CNT,W
BTFSC STATUS,ZF
GOTO DISP1
MOVLW .1
SUBWF DISP_CNT,W
BTFSC STATUS,ZF
GOTO DISP2
MOVLW .2
SUBWF DISP_CNT,W
BTFSC STATUS,ZF
GOTO DISP3
MOVLW .3
SUBWF DISP_CNT,W
BTFSC STATUS,ZF
GOTO DISP4
RETURN

DISP1
MOVLW .0
SUBWF D_MODE,W
BTFSC STATUS,ZF
CALL DISP_MIN1

MOVLW .1
SUBWF D_MODE,W
BTFSC STATUS,ZF
CALL DISP1_SEC1

GOTO DISP_END

DISP2
MOVLW .0
SUBWF D_MODE,W
BTFSC STATUS,ZF 
CALL DISP_MIN10

MOVLW .1
SUBWF D_MODE,W
BTFSC STATUS,ZF
CALL DISP1_SEC10
INCF INT_CNT

GOTO DISP_END

DISP3
MOVLW .0
SUBWF D_MODE,W
BTFSC STATUS,ZF
CALL DISP_HOUR1

MOVLW .1
SUBWF D_MODE,W
BTFSC STATUS,ZF
CALL DISP1_MIN1

GOTO DISP_END

DISP4
MOVLW .0
SUBWF D_MODE,W
BTFSC STATUS,ZF
CALL DISP_HOUR10

MOVLW .1
SUBWF D_MODE,W
BTFSC STATUS,ZF
CALL DISP1_MIN10
INCF INT_CNT

GOTO DISP_END

DISP_END
INCF INT_DNT
INCF DISP_CNT
MOVLW .4
SUBWF DISP_CNT,W
BTFSC STATUS,ZF
CLRF DISP_CNT
RETURN

; main program 시작
START_UP
BSF STATUS,RP0 ; BANK 1 선택
MOVLW B'00001111' 
MOVWF TRISA
MOVLW B'00000000'
MOVWF TRISB
MOVLW B'00000000'
MOVWF TRISC
MOVLW B'00000111'
MOVWF ADCON1

; INTERRUPT 시간 설정 --- 2.048msec 주기
MOVLW B'00000010' ; 2.048msec (1:8)
MOVWF OPTION_REG
BCF STATUS,RP0 ; RAM BANK 0 선택
BSF INTCON,5 ; TIMER INTERRUPT ENABLE
BSF INTCON,7 ; GLOBAL INT. ENABLE

MAIN_ST
; 여기서부터 USER PROGRAM 작성
; 변수 초기화
CLRF INT_CNT
CLRF D_10SEC
CLRF D_1SEC
CLRF DISP_CNT
CLRF D_MIN1
CLRF D_MIN10
CLRF D_HOUR1
CLRF D_HOUR10
CLRF D_MODE
CLRF D_DOT
CLRF INT_DNT
CLRF OPT_MODE
CLRF AMPM

M_LOOP ;-----------------MAIN-----------------------
BTFSS PORTA,0 
CALL SWITCH
BTFSS PORTA,2
CALL SWITCH_3
MOVLW .242	;0.5초
SUBWF INT_DNT,W	;0.5초단위 변수 overflow마다 2회마다 1씩증가
BTFSC STATUS,ZF
CALL DOT_AD
MOVLW .242 ; 수정하면 정확도 조정가능  //250
SUBWF INT_CNT,W ;1초단위 변수 overflow마다 2회마다 1씩증가
BTFSS STATUS,ZF
GOTO M_LOOP

; 1sec 마다 들어오는 부분
CK_LOOP
CLRF INT_CNT ; 다음 1초를 기다리기 위한 초기화 INT_CNT=0
INCF D_1SEC ; 1초 단위 변수 증가
MOVLW .10
SUBWF D_1SEC,W
BTFSS STATUS,ZF
GOTO M_LOOP

; 10초마다 들어오는 부분
CLRF D_1SEC ; 다음 10초를 기다리기 위한 초기화 D_1SEC=0
INCF D_10SEC ; 10초 단위 변수 증가
MOVLW .6 ; 수정하면 표시숫자 범위 수정가능  //6
SUBWF D_10SEC,W
BTFSC STATUS,ZF
CALL MIN1 
GOTO M_LOOP

MIN1 ;1분마다 들어오는 부분
CLRF D_10SEC 
INCF D_MIN1
MOVLW .10
SUBWF D_MIN1,W
BTFSS STATUS,ZF
GOTO M_LOOP

;10분마다 들어오는 부분
CLRF D_MIN1
INCF D_MIN10
MOVLW .6
SUBWF D_MIN10,W
BTFSC STATUS,ZF
CALL HOUR1
RETURN

HOUR1
CLRF D_MIN10
INCF D_HOUR1
MOVLW .2
SUBWF D_HOUR1,W
BTFSC STATUS,ZF
CALL HOUR2
MOVLW .10
SUBWF D_HOUR1,W
BTFSC STATUS,ZF
GOTO HOUR4
RETURN

HOUR2
MOVLW .0
SUBWF D_HOUR10,W
BTFSS STATUS,ZF
GOTO HOUR3
RETURN

HOUR3
INCF AMPM
CLRF D_HOUR1
CLRF D_HOUR10
RETURN

HOUR4
CLRF D_HOUR1
INCF D_HOUR10
RETURN

;SUBROUTINE 1 CONV
 CONV ANDLW B'00001111' ; W의 low nibble 값을 변환하자.
 ADDWF PCL,F ; PCL+변환 숫자값 --> PCL
 RETLW B'11111100' ;'0'를 표시 하는 값이 W로 들어옴
 RETLW B'01100000' ; '1'를 표시 하는 값
 RETLW B'11011010' ; '2'를 표시 하는 값
 RETLW B'11110010' ; '3'를 표시 하는 값
 RETLW B'01100110' ; '4'를 표시 하는 값
 RETLW B'10110110' ; '5'를 표시 하는 값
 RETLW B'10111110' ; '6'를 표시 하는 값
 RETLW B'11100100' ; '7'를 표시 하는 값
 RETLW B'11111110' ; '8'를 표시 하는 값
 RETLW B'11110110' ; '9'를 표시 하는 값
 RETLW B'11101110' ; 'A'를 표시 하는 값
 RETLW B'00111110' ; 'b'를 표시 하는 값
 RETLW B'10011100' ; 'C'를 표시 하는 값
 RETLW B'01111010' ; 'd'를 표시 하는 값
 RETLW B'10011110' ; 'E'를 표시 하는 값
 RETLW B'10001110' ; 'F'를 표시 하는 값
 
 
 ;기본 디스플레이 모드 @@@@@@@@@@@@@@@@@@@@@@@@@@@
 DISP_MIN1
 MOVF D_MIN1,W
 CALL CONV

BTFSC OPT_MODE,0
 CALL DOT_ON

 MOVWF PORTC
 MOVLW B'00001110'
 CALL LED
 MOVWF PORTB
 RETURN
 
  DISP_MIN10
 MOVF D_MIN10,W
 CALL CONV
 
 BTFSC OPT_MODE,0
 CALL DOT_ON
 
 MOVWF PORTC
 MOVLW B'00001101'
 CALL LED
 MOVWF PORTB
 RETURN
 
 DISP_HOUR1
 MOVF D_HOUR1,W
 CALL CONV
 BTFSC OPT_MODE,0
 CALL DOT_ON
 BTFSS D_DOT,0
 IORLW B'00000001'
 MOVWF PORTC
 MOVLW B'00001011'
 CALL LED
 MOVWF PORTB
 RETURN
 
 DISP_HOUR10
 MOVF D_HOUR10,W
 CALL CONV
 
 BTFSC OPT_MODE,0
 CALL DOT_ON
 
 MOVWF PORTC
 MOVLW B'00000111'
 CALL LED
 MOVWF PORTB
 RETURN
 
 
 
 ;스위치 1 초 표시 모드 @@@@@@@@@@@@@@@@@@@@@
 DISP1_SEC1
 MOVF D_1SEC,W
 CALL CONV
 MOVWF PORTC
 MOVLW B'00001110'
 CALL LED
 MOVWF PORTB
 RETURN

 DISP1_SEC10
 MOVF D_10SEC,W
 CALL CONV
 MOVWF PORTC
 MOVLW B'00001101'
 CALL LED
 MOVWF PORTB
 RETURN
 
 DISP1_MIN1
 MOVF D_MIN1,W
 CALL CONV
 
 ANDLW B'00000000'
 BTFSC D_DOT,0
 IORLW B'00000001'
 MOVWF PORTC
 MOVLW B'00001011'
 CALL LED
 MOVWF PORTB
 RETURN

 DISP1_MIN10
 MOVF D_MIN10,W
 CALL CONV
 MOVWF PORTC
 MOVLW B'00001111'
 CALL LED
 MOVWF PORTB
 RETURN
 
 
 SWITCH
 CALL DELAY
 CALL DELAY
 INCF D_MODE
 MOVLW .2
 SUBWF D_MODE,W
 BTFSC STATUS,ZF
 CLRF D_MODE
 
 RETURN
 
DELAY
MOVLW .225
MOVWF DBUF1 ; 긴 시간을 설정하기 위한 변수
LP1 MOVLW .200
MOVWF DBUF2
LP2 NOP
DECFSZ DBUF2, F
GOTO LP2 ; ZERO가 아니면 GOTO LP2 수행
DECFSZ DBUF1, F ;변수를 감소시켜 가면서 00이 되었나 확인
GOTO LP1 ; ZERO가 아니면 GOTO LP1수행
RETURN 

DOT_AD
INCF D_DOT
CLRF INT_DNT
RETURN
 
 SWITCH_3 
 
 CALL DELAY
 CALL DELAY
 INCF OPT_MODE 
CLRF D_10SEC
CLRF D_1SEC
CLRF D_MODE

OPT_TIME
BTFSS PORTA,0 
CALL OPT_HOUR
BTFSS PORTA,1
CALL OPT_MINUTE
BTFSS PORTA,2
GOTO OPT_END
GOTO OPT_TIME

OPT_END
CALL DELAY
CALL DELAY
CLRF OPT_MODE ;OPT_MODE 0으로
CLRF INT_CNT ; INT_CNT 0으로
CLRF INT_DNT ; INT_DNT 0으로
GOTO M_LOOP
 
 
DOT_ON
IORLW B'00000001'
RETURN

OPT_MINUTE
CALL DELAY
INCF D_MIN1; 1분 단위 변수 증가
MOVLW .10
SUBWF D_MIN1,W
BTFSS STATUS,ZF
RETURN
CLRF D_MIN1 ; 
INCF D_MIN10 ; 10분 단위 변수 증가
MOVLW .6 ; 수정하면 표시숫자 범위 수정가능  //6
SUBWF D_MIN10,W
BTFSC STATUS,ZF
CLRF D_MIN10
RETURN

OPT_HOUR
 CALL DELAY
INCF D_HOUR1

MOVLW .2
SUBWF D_HOUR1,W
BTFSC STATUS,ZF
CALL OPT_HOUR2
MOVLW .10
SUBWF D_HOUR1,W
BTFSC STATUS,ZF
GOTO OPT_HOUR4
RETURN

OPT_HOUR2
MOVLW .0 
SUBWF D_HOUR10,W
BTFSS STATUS,ZF
GOTO OPT_HOUR3
RETURN

OPT_HOUR3
INCF AMPM
CLRF D_HOUR1
CLRF D_HOUR10
RETURN

OPT_HOUR4
CLRF D_HOUR1
INCF D_HOUR10
RETURN
 
LED
BTFSC AMPM,0
IORLW B'00010000'
BTFSS AMPM,0
ANDLW B'11101111'
RETURN

 END

```
