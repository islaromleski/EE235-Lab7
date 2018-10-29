; Author : Matthew Romleski
; Tech ID: 12676184
; Program that tests a 16-bit integer by pushing it into a subroutine that:
; 1.) splits the decimal into two equal halves (for example, 12 and 33).
; 2.) Calculate the squares of these two halves.
; 3.) Add these squares together.
; 4.) If the result is the same as the original number (1233) then return 1. Else return 0.
; Then we use this subroutine to test all numbers from 1000-9999. If the return was 1, it stores this number in data memory.

		.include <atxmega128a1udef.inc>

		.dseg
		
		.def	rtrnVal = r25
		
		.def	resLow	= r6
		.def	resHi	= r7
		.def	numLow	= r16
		.def	numHi	= r17
		.def	hundLow	= r18
		.def	hundHi	= r19
		.def	temp	= r20

		.def	tempLow	= r0
		.def	tempHi	= r1
		.def	remLow	= r21
		.def	remHi	= r22
		.def	qLow	= r23
		.def	qHi		= r24

		.def	lpCntLo = r30
		.def	lpCntHi = r31

		.cseg
		.org	0x00
		rjmp	start
		.org	0xF6


start:	ldi		hundLow, 100 ; Loads 100, which we'll be using as the divisor.
		ldi		hundHi, 0 ; ^^
		
		ldi		temp, low(RAMEND) ; Initializes the stack pointer.
		out		CPU_SPL, temp ; ^^
		ldi		temp, high(RAMEND) ; ^^
		out		CPU_SPH, temp ; ^^

		ldi		XL, 0x00 ; Loads the data memory address we'll be storing info in.
		ldi		XH, 0x20 ; ^^

		call	tstrng

done:	rjmp	done


tstrng:	ldi		lpCntHi, 0x03 ; Loads the loop condition for testrange.
		ldi		lpCntLo, 0xE8 ; ^^

lpCon:	cpi		LpCntHi, 0x27 ; Checks if the high register is equal to the number for 10000.
		brne	tstloop ; Branches if it isn't.
		cpi		LpCntLo, 0x10 ; Checks if the low register is equal to the number for 10000.
		brne	tstloop ; Branches if it isn't.
		
		rjmp	rtrn ; Otherwise, we return from the subroutine.

tstloop:mov		numLow, lpCntLo
		mov		numHi, lpCntHi

		call	tester ; Returns its result in r25 (rtrnVal).

		cpi		rtrnVal, 1 ; Checks if the value is true.
		breq	store ; Stores if it is.

incLp:	ldi		temp, 1
		add		lpCntLo, temp ; Increments the loop counter.
		ldi		temp, 0
		adc		lpCntHi, temp ; ^^

		clr		rtrnVal

		jmp		lpCon

store:	st		X+, LpCntHi ; Stores the value.
		st		X+, LpCntLo ; ^^

		jmp		incLp

rtrn:	ret


tester:	push	numLow ; Stores variables in the stack.
		push	numHi ; ^^
		push	hundLow ; ^^
		push	hundHi ; ^^
		
		call	div16U ; Result stored in r23 (qLow), remainder in r21 (remLow). Q=12, R=33
		
		pop		hundHi ; Pops variables from the stack.
		pop		hundLow ; ^^
		pop		numHi ; ^^
		pop		numLow ; ^^
		
		mul		qLow, qLow ; Calculates 12^2.
		mov		resLow, tempLow ; Moves the result of the multiplication.
		mov		resHi, tempHi ; ^^
		mul		remLow, remLow ; Calculates 33^2.
		add		resLow, tempLow ; Adds 12^2 & 33^2.
		adc		resHi, tempHi ; ^^
		
		cp		resLow, numLow ; Compares the low bits of the result with the low bits of the original number.
		brne	rtrn0
		cp		resHi, numHi ; ^^ high bits ^^ high bits of the original number.
		brne	rtrn0

rtrn1:	ldi		rtrnVal, 1 ; Only does this is resLow=numLow and resHi=numHi.
		jmp		return

rtrn0:	ldi		rtrnVal, 0 ; Does this is resLow=/=numLow or resHi=/=numHi.

return:	ret


div16U:	ldi		temp, 0 ; Resets the loop counter.
		mov		qLow, numLow ; Sets numLow:numHi as the dividend.
		mov		qHi, numHi ; ^^
		clr		remLow ; Sets remainder registers to zero.
		clr		remHi ; ^^
divlp:	lsl		qLow ; Shifts left, stores MSB in carry.
		rol		qHi ; Rotates carry into lowest, MSB out to carry.
		rol		remLow ; ^^
		rol		remHi ; ^^
		mov		tempLow, remLow ; Saves the remainder to a temp registry.
		mov		tempHi, remHi ; ^^
		sub		tempLow, hundLow ; Subtracts our divisor from the remainder.
		sbc		tempHi, hundHi ; ^^
		brmi	negtve ; Branches if negative.
		ori		qLow, 0x01 ; Sets 0 bit in the Quotient to 1 (if result of SBC was positive).
		mov		remLow, tempLow ; Updates the remainder.
		mov		remHi, tempHi ; ^^
		jmp		incrmt ; 
negtve: andi	qLow, 0xFE ; Sets 0 bit in Quotient to 0 (if result of SBC was negative).
incrmt:	inc		temp ; Increments the loop counter.
		cpi		temp, 16 ; Sees if the subroutine has looped 16 times.
		brne	divlp ; If it hasn't, loops again.
		ret		; Otherwise it, exits from the subroutine.