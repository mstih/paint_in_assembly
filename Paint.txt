JMP main
;########################################
;VARIABLES
;########################################

;colors(b:blue,r:red,g:green,y=yellow, o=orange)
txt_color: DB 255 ;(white)
current_color: DB 11
blue: DB 11
red: DB 224
green: DB 24
yellow: DB 252
orange: DB 244

;positions of the current pixel
curr_x: DB 0
curr_y: DB 0

;text for keyboard display
disp: DB "Color:"
	  DB 0

;menu text variables
menu1: DB "Welcome to"
	   DB 0             
menu2: DB "PAINT"
       DB 0
menu3: DB "Press SPACE to"
	   DB 0
menu4: DB "continue"
       DB 0

;Instruction text variables
ins1: DB "INSTRUCTIONS:"
	  DB 0
ins2: DB "PAINT: SPACE"
      DB 0 
ins3: DB "COLORS:b r g y o"
      DB 0
ins4: DB "MOVE: w a s d"
	  DB 0
ins5: DB "QUIT: q"
	  DB 0
ins6: DB "SPACE to Start"
	  DB 0
      
;Goodbye screen
final1: DB "Thanks for using"
	    DB 0
final2: DB "PAINT!"
	    DB 0
;########################################
;~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
;########################################

;########################################
;print_text() 
; 	PARAM: 
;	- reg C (String pointer)
; 	- reg D (Display location)			  
;########################################
print_text:
print_text_loop:
	MOV A, D				;Pointer to VIDADDR
    OUT 8					
    MOVB AH, [C]			;first letter on AH
    CMPB AH, 0				;check for '\0'
    JE print_text_return				
    MOVB AL, [txt_color]	;color to AL
    OUT 9					;send A to VIDDATA
    INC C
   	ADD D, 2				
    JMP print_text_loop
print_text_return:
RET
;########################################
;~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
;########################################

;########################################
;menu_pooling() - simple space pooling
;########################################
menu_pooling:
    pooling:
    		IN 5					;get status
            CMP A, 0				;nothing? -> repeat
            JE pooling			
            AND A, 2	
            CMP A, 2				;is it keyup?
            JE key_pressed
            IN 6					;reset status
            JMP pooling
    key_pressed:
    		IN 6
           	CMP A, 0x20				;is it spacebar?
            JNE pooling			;no? -> then wait for response again
menu_pooling_return:            
	RET    
;########################################
;~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
;########################################    
     
;########################################
;draw_menu() - draws a menu screen
;########################################
draw_menu:
	MOV A, 1    			;GPU text mode
    OUT 7					;VIDMODE text
    
    ;writing "Welcome to"
    MOV C, menu1			;String pointer
    MOV D, 774		        ;Screen location pointer
	CALL print_text
    
	;writing "PAINT" -- simmilar to the loop above
    MOV C, menu2
    MOV D, 1290
	CALL print_text
    
	;WRITING "-" -- hardcoded "-", no variable
    MOV C, 45				;ASCII for '-'
    MOV D, 2062				;Pointer to next line in the middle
    MOV A, D
    OUT 8
    MOVB AH, CL				;No loop? --> no checking for 0
    MOVB AL, [txt_color]			
    OUT 9
    
	;writing "Press SPACE"
    MOV C, menu3
    MOV D, 2818
	CALL print_text  
    
	;writing "Continue"
    MOV C, menu4
    MOV D, 3080
    CALL print_text
    
    ;Pooling for response from user( Space pressed )
    CALL menu_pooling
RET
;#########################################
;~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
;#########################################


;#########################################
;draw_instructions() - draws instruction 
;on which keys do what
;#########################################
draw_instructions:
	;clear display from previous text menu
    MOV A, 3
    OUT 7
    
	;writing "INSTRUCTIONS"
    MOV C, ins1				;pointer to first char in string
    MOV D, 0				;pointer to display address
    CALL print_text
    
	;writing first instruction "PAINT"
    MOV C, ins2
    MOV D, 768				
    CALL print_text
	
	;writing instruction2 "Color"
    MOV C, ins3
    MOV D, 1280
    CALL print_text

	;writing instruction3 "Exit"
    MOV C, ins4
    MOV D, 1792
    CALL print_text
    
	;writing instruction4 
    MOV C, ins5
    MOV D, 2304
    CALL print_text

	;writing "Space to start"
	MOV C, ins6
    MOV D, 3586
    CALL print_text

	;pooling for space to begin game
	CALL menu_pooling
    RET
;#########################################
;~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
;#########################################


;#########################################
; square_paint() - paints a square with the current
; color and stores location for next one
;#########################################
square_paint:
	MOVB BH, [curr_y]		;current y location
    MOVB BL, [curr_x]		;current x location
    MOV D, 0				;counter for rows in a square
	sq_loop:
    	MOV C, 0			;counter for pixels in row
        ;inner loop that paints pixels in a row
    	sq_loop2:
			MOV A, B					;send location of pixel to vidaddr
            OUT 8		
			MOVB AL, [current_color]	;set color to pixel
            OUT 9
            INC C
            CMP C, 16					;whole line (16px) already? -> then next line
            JE sq_loop2_return
            INCB BL						;position x = x+1
            JMP sq_loop2
        sq_loop2_return:
        	INC D						
        	CMP D, 16					;do i have enough rows -> if yes stop painting
            JE sq_loop1_return
        	MOVB BL, [curr_x]
        	INCB BH
            JMP sq_loop     
	sq_loop1_return:
            ;move location to the first pixel in the next square (position 0,0 right of the currently drawn square)
            SUBB BH, 15					;moves it back to top
            SUBB BL, 15					;moves one square right
            ;stores the values after each time painting a square 
    		MOVB [curr_y], BH			;stores the y of the first pixel for next time drawing			
    		MOVB [curr_x], BL  			;stores the x of the first pixel for next time drawing
square_function_return:
		RET
;#####################################################################
;~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
;#####################################################################

;#####################################################################
;main label - 
;#####################################################################
main:
	MOV SP, 0x0FFF                 ;initialises stack pointer for functions
    CALL draw_menu                 ;draws a menu and starts menu pooling
    CALL draw_instructions         ;draws instructions after menu returns and again pooling
;MAIN PROGRAM - after the two menu
	MOV A, 2	;changes display to BITMAP MODE
    OUT 7		;sends that to VIDMODE I/O reg.
	MOV A, 3	;resets the display
    OUT 7
	;writes a "Color:" on small display
	MOV A, disp					;pointer on first char
    MOV D, 0x1000				;pointer on first display location
	type_loop:
       	MOVB BL, [A]
        CMPB BL, 0				;is the string over?
        JE type_loop_return
        MOVB [D], BL
        INC D					;next display addr
        INC A					;next char
   		JMP type_loop
    type_loop_return:
    ;default color is blue
	MOVB DL, [blue]
    MOVB [current_color], DL
    MOVB [0x1006], 'B'

;main pooling for keyboard in bitmap mode
game_pooling:
	IN 5					;read kbdstatus
    CMP A, 0
    JE game_pooling			;if nothing keep looping
    MOV B, A
    AND B, 2
    CMP B, 2				;keyup event? -> check what key
    JE game_key_pressed
	IN 6					;anything but keyup -> resets KBDSTATUS
    JMP game_pooling

;if key was pressed, check which one was it
game_key_pressed:
	IN 6
    CMPB AL, 0x71			
    JE exit_program			;"q"(ascii: 71) pressed? -> exit the game (final screen)
    CMPB AL, 0x20
    JE paint				;"space"(ascii: 20) pressed? -> paint next square
    CMPB AL, 0x62
    JE blue_pressed			;"b"(ascii: 62) pressed? -> change color
    CMPB AL, 0x72			
    JE red_pressed			;"r"(ascii: 72) pressed? -> change color
    CMPB AL, 0x79
    JE yellow_pressed		;"y"(ascii: 79) pressed? -> change color
    CMPB AL, 0x67	
    JE green_pressed		;"g"(ascii: 67) pressed? -> change color
    CMPB AL, 0x6F
    JE orange_pressed		;"0"(ascii: 6F) pressed? -> change color
    CMPB AL, 0x77			
    JE up_pressed			;"w"(ascii: 77) pressed? -> move one square up
    CMPB AL, 0x73
    JE down_pressed			;"s"(ascii: 73) pressed? -> move one square down
    CMPB AL, 0x61			
    JE left_pressed			;"a"(ascii: 61) pressed? -> move one square left
    CMPB AL, 0x64
    JE right_pressed		;"d"(ascii: 64) pressed? -> move one right
    JMP game_pooling
    
paint:
	CALL square_paint
    JMP game_pooling

;actions for changing current_color
blue_pressed:
	MOVB DL, [blue]				;moves code for blue in DL
    MOVB [current_color], DL	;changes current_color to be selected color
    MOVB [0x1006], 'B'			;moves char to display
    JMP game_pooling			
red_pressed:
	MOVB DL, [red]				;read red color
    MOVB [current_color], DL	;move red color to be current
    MOVB [0x1006], 'R'			;replace current color char to 'R'
    JMP game_pooling			
yellow_pressed:
	MOVB DL, [yellow]			;read yellow color
    MOVB [current_color], DL	;move yellow color to be current color
    MOVB [0x1006], 'Y'			;set char to be 'Y'
    JMP game_pooling
green_pressed:				
	MOVB DL, [green]			;read green color
    MOVB [current_color], DL	;move it to current color
    MOVB [0x1006], 'G'			;set char to be 'G'
    JMP game_pooling
orange_pressed:
	MOVB DL, [orange]			;read orange color
    MOVB [current_color], DL	;move it to current color
    MOVB [0x1006], 'O'			;set char to be 'O'
    JMP game_pooling
    
;actions for moving up/down/left/right
up_pressed:
	MOVB BH, [curr_y]		;moves y position into BH
    CMPB BH, 0				;checks if already at the top
    JE game_pooling			;if yes, do nothing
    SUBB BH, 16				;if not at the top, move y position for one square up(16px) 
    MOVB [curr_y], BH       ;store value back to label
    JMP game_pooling		;back to pooling
down_pressed:
	MOVB BH, [curr_y]
    CMPB BH, 240			;compare if not on the last row
    JE game_pooling
    ADDB BH, 16				;otherwise add 16px to y
    MOVB [curr_y], BH
    JMP game_pooling
left_pressed:
	MOVB BL, [curr_x]
    CMPB BL, 0				;compare if on the left edge
    JE game_pooling	
    SUBB BL, 16				;if not move left
    MOVB [curr_x], BL
    JMP game_pooling
right_pressed:
	MOVB BL, [curr_x]
    CMPB BL, 240			;compare if already on the right edge
    JE game_pooling
    ADDB BL, 16				;if not add 16px to x
    MOVB [curr_x], BL
    JMP game_pooling
   
;######################################################################
;~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
;######################################################################

;stops running the game when pressed
exit_program:
	;MOV A, 3		;resets the screen
    ;OUT 7
    MOV A, 1		;back to text mode
    OUT 7
    
    ;deleting small screen
    MOV B, 10
    MOV C, 0x1000
   	delete_loop:
    	MOV [C], 0
    	DEC B
        INC C
    	CMP B, 0
        JNE delete_loop			
    
    ;writing final text
	MOV C, final1
    MOV D, 0x0606 				;0x0606 or 1542
    CALL print_text
    MOV C, final2
    MOV D, 0x090A				;0x090A or 2314
    CALL print_text
    
	HLT