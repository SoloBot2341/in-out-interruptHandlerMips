	.data
source:		.asciiz "James went to the store and spent $5._He left with $101..()\n"
display: 	.space 61
canQuit:  	.word 0
	.text
	.globl main
main:
	addi	$sp, $sp -4
	la	$t0, source
	sw	$t0, ($sp)
	addi	$sp, $sp, -4
	la	$t0, display
	sw	$t0, ($sp)
	jal	copyRoutine # copy the source string into the display.
	nop
	addi	$sp, $sp, 8		# pop the arguments

	mfc0	$s0, $12		# load the status register
	ori 	$s0, $s0, 0xC01		# mask to enable needed interrupts.
	mtc0 	$s0, $12		# store the new value 

	or  $s1, $zero, 0xffff0000    	# Load upper half of the Receiver Control Register address
    	ori $t1, $zero, 0x02      	# 0x02
    	sw  $t1, 0($s1)           	# Set the Receiver Control Register
    	sw  $t1, 8($s1)           	# Set the Transmitter Control Register
loop:
	lw	$s2, canQuit
	beqz	$s2, loop
end:
	li	$v0, 10
	syscall

copyRoutine:				# routine to copy source into display
	addi	$sp, $sp, -4
	sw	$ra, ($sp)		# push the return address
	addi	$sp, $sp, -4	
	sw	$fp, ($sp)		# push the frame pointer
	
	lw	$a1, 8($sp)		# get the second argument
	lw	$a0, 12($sp)		# get the first argument
copyLoop:
	lb	$t0, 0($a0)
	sb	$t0, 0($a1)
	beqz	$t0, doneCopying
	addi	$a0, 1
	addi	$a1, 1
	j	copyLoop
doneCopying:
	lw	$fp, ($sp)	# pop the frame pointer
	addi	$sp, $sp, 4
	lw	$ra, ($sp)	# pop the return address
	addi	$sp, $sp, 4
	jal	$ra

	.kdata
saved_at:	.word 0
saved_s0:	.word 0
saved_s1:	.word 0
saved_a1:	.word 0
saved_t0:	.word 0
saved_t1:	.word 0
saved_t2:	.word 0
saved_t3:	.word 0 
saved_t4:	.word 0
saved_t5:	.word 0
strIndex:	.word 0
	.ktext 0x80000180 		# exception handler begins here.
exceptionHandler:
	.set	noat         		# Disable pseudo-instruction use of $at
	move	$k1, $at     		# Move $at to $k1
	sw	$k1, saved_at  		# Save $k1 into 'saved_at'
	.set	at           		# Re-enable pseudo-instruction use of $at
		
	sw	$s0, saved_s0		# save $s0
	sw	$s1, saved_s1		# save $s1
	sw	$t0, saved_t0		# save $t0
	sw	$t1, saved_t1		# save $t1
	sw	$t2, saved_t2		# save $t2
	sw	$t3, saved_t3		# save $t3
	sw	$t4, saved_t4		# save $t4
	
	mfc0	$t5, $13 		# Copy Cause register value to $t5.
	andi	$t4, $t5, 0x7C		# exception code mask.
	bnez	$t4, return		# interrupt didn't occur

getInput:	
	andi 	$t4, $t5, 0x400		# value of the 11th bit in the cause reg.
	bnez	$t4, canPrint		# not a reciever interrupt  	
	lw    	$t4, 4($s1)		# $v0 = inputted char   

	beq	$t4, 115, sortIt	# Modify the display string if needed.	
	beq	$t4, 97, copyIt
	beq	$t4, 114, reverseIt
	beq	$t4, 116, toggleIt
	bne	$t4, 113, canPrint
	li	$t4, 1
	sw	$t4, canQuit
	j	return
canPrint:
	la	$a1, display		# load the display string
	lw	$k1, strIndex 		# load the index of the current char to print.
	add	$a1, $a1, $k1		# point the display to the correct address.
	and 	$t4, $t5, 0x200		# value of 10th bit in the cause reg
	bnez	$t4, return		# not a transmitter interrupt
	lb	$t4, 0($a1)		# load next character of string
	sw	$t4, 12($s1)		# write character to the console
	beq	$t4, 10, reset		# reset if $t0 == new line
	lw	$t4, strIndex	
	addi	$t4, 1
	sw	$t4, strIndex		# increment the string index
	j	return
reset:					
	sw	$zero, strIndex		# point back to the base address
return:
	lw	$s0, saved_s0
	lw	$s1, saved_s1 
	lw	$t0, saved_t0
	lw	$t1, saved_t1
	lw	$t2, saved_t2
	lw	$t3, saved_t3
	lw	$t4, saved_t4
	lw	$t5, saved_t5
	mtc0	$s0, $12
	.set 	noat         		# Disable pseudo-instruction use of $at
	lw	$k1, saved_at  		# Load the saved value of $at from memory into $k1
	move	$at, $k1     		# Restore $at from $k1
	.set	at           		# Re-enable pseudo-instruction use of $at
	eret
reverseIt:
    	la	$t4, display      
    	addi	$t1, $t4, 58 		# last character before "\n"
reverseLoop:
    	bge	$t4, $t1, canPrint 	
    	lb	$t2, 0($t4)                 	
    	lb	$t3, 0($t1)                  	
    	sb 	$t2, 0($t1)                  	
    	sb 	$t3, 0($t4)                  	
    	addi	$t4, 1                
    	addi	$t1, -1                
    	j	reverseLoop                  	
toggleIt:
	la	$t4, display
toggleLoop:
	lb	$t0, 0($t4)
	beqz	$t0, canPrint

	sle	$t1, $t0, 90		# Check if $t0 than or equal to 'Z'
	sge	$t2, $t0, 65		# Check if $t0 greater than or equal to 'A'
	and	$t1, $t2, $t1		# upper case check

	sge	$t2, $t0, 97		# Check if $t0 greater than or equal to 'a'
	sle	$t3, $t0, 122		# Check if $t0 less than or equal to 'z'
	and	$t3, $t2, $t3		# lower case check

	or	$t3, $t1, $t3		# Combine checks
	beqz	$t3, nextChar		# If not a letter, skip toggling
	xori	$t0, $t0, 32		# (a xor b) xor b = a
	sb	$t0, 0($t4)
nextChar:
	addi	$t4, $t4, 1
	j	toggleLoop

sortIt: 				# bubble sort
	la	$t4, display
	add	$t0, $t4, 59 		# address of the endline char.
l1:					# loop through characters from right to left.
	addi	$t0, -1			
	add	$t1, $t4, -1 		
	lb	$t3, 0($t0)		
	blt	$t0, $t4, canPrint	
l2:					# loop through characters from base addr to $t1
	add	$t1, 1			
	lb	$t2, 0($t1)		
	beq	$t1, $t0, l1		 
	blt	$t3, $t2, swap		# compare the elements and swap if needed.
	j	l2
swap:
	sb	$t3, 0($t1)
	sb	$t2, 0($t0)
	lb	$t3, 0($t0)	
	j	l2

copyIt: # code block to copy source into display
	la	$t1, source
	la	$t2, display
cLoop:
	lb	$t0, 0($t1)
	sb	$t0, 0($t2)
	beqz	$t0, canPrint
	addi	$t1, 1
	addi	$t2, 1
	j	cLoop
