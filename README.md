# Interfaz-keypad-1-x1-7-segment-display
#include <stdint.h>
#include "stm32l053xx.h"

#if !defined(__SOFT_FP__) && defined(__ARM_FP)
  #warning "FPU is not initialized, but the project is compiling for an FPU. Please initialize the FPU before use."
#endif

void keyboard_config();
void print_bcd_7_segment_decoder_CC(uint8_t);
void delayMs(int n);
void key_pressed();
void init();
uint8_t last_displayed_value = 0;  // variable donde me guarda el valor presionado actual

int main(void){

	init();

	while(1){
		keyboard_config();		//SE MANDA A LLAMAR LA FUNCION QUE TIENE LA CONFIGURACION DE LAS FILAS

		key_pressed();			//SE MANDA A LLAMAR LA FUNCION QUE TIENE LA CONFIGURACION DEL MULTIPLEXADO DE COLUMNAS E IMPRESION EN LOS DISPLAYS
	}

}


void init(){

	RCC->IOPENR |=0x01<<0;		//HABILITA EL REGISTRO DE RELOJ DEL PUERTO GPIOA
	RCC->IOPENR |=0x01<<1;		//HABILITA EL REGISTRO DE RELOJ DEL PUERTO GPIOB
	RCC->IOPENR |=0x01<<2;		//HABILITA EL REGISTRO DE RELOJ DEL PUERTO GPIOC



	GPIOB->MODER &=~(1<<17);	//PB8 COMO SALIDA CON EL SEGMENTO A
	GPIOB->MODER &=~(1<<5);		//PB2 COMO SALIDA CON EL SEGMENTO B
	GPIOB->MODER &=~(1<<7);		//PB3 COMO SALIDA CON EL SEGMENTO C
	GPIOB->MODER &=~(1<<9);		//PB4 COMO SALIDA CON EL SEGMENTO D
	GPIOB->MODER &=~(1<<13);	//PB6 COMO SALIDA CON EL SEGMENTO E
	GPIOB->MODER &=~(1<<15);	//PB7 COMO SALIDA CON EL SEGMENTO F
	GPIOB->MODER &=~(1<<19);	//PB9 COMO SALIDA CON EL SEGMENTO H

	GPIOA->MODER &=~(1<<1);		//PA0 COMO HABILITADOR DEL COMUN DEL DISPLAY 0
	GPIOA->MODER &=~(1<<3);		//PA1 COMO HABILITADOR DEL COMUN DEL DISPLAY 1

	GPIOA->BSRR = (0x03<<16);	//INICIA COLOCANDO 0 LOS PUERTOS PA0 Y PA1


}
void keyboard_config(){

	//CONFIGURACION DE LAS FILAS
	//CPNFIGURACION DE PC5
	GPIOC->MODER &=~(1<<10);	//SE HABILITA COMO ENTRADA
	GPIOC->MODER &=~(1<<11);
	GPIOC->PUPDR |=(1<<10);		    //SE HABILITA EL PULL-UP DE PC5
	//CONFIGURACION DE PC6
	GPIOC->MODER &=~(1<<12);	//SE HABILITA COMO ENTRADA
	GPIOC->MODER &=~(1<<13);
	GPIOC->PUPDR |=(1<<12);		    //SE HABILITA EL PULL-UP DE PC6
	//CONFIGURACION DE PC8
	GPIOC->MODER &=~(1<<16);	//SE HABILITA COMO ENTRADA
	GPIOC->MODER &=~(1<<17);
	GPIOC->PUPDR |=(1<<16);		     //SE HABILITA EL PULL-UP DE PC8
	//CONFIGURACION DE PC9
	GPIOC->MODER &=~(1<<18);	//SE HABILITA COMO ENTRADA
	GPIOC->MODER &=~(1<<19);
	GPIOC->PUPDR |=(1<<18);		    //SE HABILITA EL PULL-UP DE PC9

	//COLUMN CONFIG
	//PB11 COL0 OUTPUT-HIGH
	GPIOB->MODER &=~(1<<23);	//SE HABILITA COMO ENTRADA
	//PC7 COL1 OUTPUT-HIGH
	GPIOC->MODER &=~(1<<15);	//SE HABILITA COMO ENTRADA
	//PA9 COL2 OUTPUT-HIGH
	GPIOA->MODER &=~(1<<19);  	//SE HABILITA COMO ENTRADA
	//PC4 COL3 OUTPUT-HIGH
	GPIOC->MODER &=~(1<<9);		//SE HABILITA COMO ENTRADA
}

void key_pressed(){

	//SCAN COL0
	GPIOB->ODR &=~(1<<11); 	//COLOCA PB11 COMO 0
	GPIOC->ODR |=(1<<7);	//COLOCA PC7 COMO 1
	GPIOA->ODR |=(1<<9);	//COLOCA PA9 COMO 1
	GPIOC->ODR |=(1<<4);	//COLOCA PC4 COMO 1
	delayMs(5);
	if(!(GPIOC->IDR & 0x20)){			//EVALUANDO FILA 0
		//SE PRESIONO LA TECLA 1

		print_bcd_7_segment_decoder_CC(0x01);	//IMPRIME EL NO. 1
		GPIOA->ODR |=(1<<1);					//HABILITA EL DISPLAY 0
		delayMs(5);								//DELAY
		//GPIOA->BSRR = (0x03<<16);				//RESETEA LOS BITS DE LOS DISPLAYS
	}else if(!(GPIOC->IDR & 0x40)){		//EVALUANDO FILA 1
		//SE PRESIONO LA TECLA 4

		print_bcd_7_segment_decoder_CC(0x04);	//IMPRIME EL NO. 4
		GPIOA->ODR |=(1<<1);					//HABILITA EL DISPLAY 0
		delayMs(5);								//DELAY
		//GPIOA->BSRR = (0x03<<16);				//RESETEA LOS BITS DE LOS DISPLAYS
	}else if(!(GPIOC->IDR & 0x100)){	//EVALUANDO FILA 2
		//SE PRESIONO LA TECLA 7

		print_bcd_7_segment_decoder_CC(0x07);
		GPIOA->ODR |=(1<<1);
		delayMs(5);
		//GPIOA->BSRR = (0x03<<16);
	}else if(!(GPIOC->IDR & 0x200)){	//EVALUANDO FILA 3
	                                                 	//SE PRESIONO LA TECLA *

		//print_bcd_7_segment_decoder_CC(0x00);
		GPIOA->BSRR=(0x3<<16);
		//GPIOA->ODR |=(1<<1);
		delayMs(5);
		//GPIOA->BSRR = (0x03<<16);
	}
	//////////////SCAN COL1
	GPIOB->ODR |=(1<<11);	//COLOCA PB11 COMO 1
	GPIOC->ODR &=~(1<<7);	//COLOCA PC7 COMO 0
	GPIOA->ODR |=(1<<9);	//COLOCA PA9 COMO 1
	GPIOC->ODR |=(1<<4);	//COLOCA PC4 COMO 1
	delayMs(5);
	if(!(GPIOC->IDR & 0x20)){			//EVALUANDO FILA 0

		print_bcd_7_segment_decoder_CC(0x02);
		GPIOA->ODR |=(1<<1);
		delayMs(5);
		//GPIOA->BSRR = (0x03<<16);
	}else if(!(GPIOC->IDR & 0x40)){		//EVALUANDO FILA 1
		//SE PRESIONO LA TECLA 5
			print_bcd_7_segment_decoder_CC(0x05);
		GPIOA->ODR |=(1<<1);
		delayMs(5);
		//GPIOA->BSRR = (0x03<<16);
	}else if(!(GPIOC->IDR & 0x100)){	//EVALUANDO FILA 2
		//SE PRESIONO LA TECLA 8

		print_bcd_7_segment_decoder_CC(0x08);
		GPIOA->ODR |=(1<<1);
		delayMs(5);
		//GPIOA->BSRR = (0x03<<16);
	}else if(!(GPIOC->IDR & 0x200)){	//EVALUANDO FILA 3
		//SE PRESIONO LA TECLA 0

		print_bcd_7_segment_decoder_CC(0x00);
		GPIOA->ODR |=(1<<1);
		delayMs(5);
		//GPIOA->BSRR = (0x03<<16);
	}
	//SCAN COL2
	GPIOB->ODR |=(1<<11);	//COLOCA PB11 COMO 1
	GPIOC->ODR |=(1<<7);	//COLOCA PC7 COMO 1
	GPIOA->ODR &=~(1<<9);	//COLOCA PA9 COMO 0
	GPIOC->ODR |=(1<<4);	//COLOCA PC4 COMO 1
	delayMs(5);
	if(!(GPIOC->IDR & 0x20)){			//EVALUANDO FILA 0
		//SE PRESIONO LA TECLA 3

		print_bcd_7_segment_decoder_CC(0x03);
		GPIOA->ODR |=(1<<1);
		delayMs(5);
		//GPIOA->BSRR = (0x03<<16);
	}else if(!(GPIOC->IDR & 0x40)){		//EVALUANDO FILA 1

		print_bcd_7_segment_decoder_CC(0x06);
		GPIOA->ODR |=(1<<1);
		delayMs(5);
		//GPIOA->BSRR = (0x03<<16);
	}else if(!(GPIOC->IDR & 0x100)){	//EVALUANDO FILA 2
		//SE PRESIONO LA TECLA 9

		print_bcd_7_segment_decoder_CC(0x09);
		GPIOA->ODR |=(1<<1);
		delayMs(5);
		//GPIOA->BSRR = (0x03<<16);
	}else if(!(GPIOC->IDR & 0x200)){	//EVALUANDO FILA 3

		 print_bcd_7_segment_decoder_CC (last_displayed_value); // Imprime el valor anterior
		        GPIOA->ODR |= (1 << 1);

		//print_bcd_7_segment_decoder_CC(0x05);
		//print_bcd_7_segment_decoder_CC(x);   /////////////////////////////////////////
		//GPIOA->ODR |=(1<<1);
		delayMs(5);
		//GPIOA->BSRR = (0x03<<16);
	}
	//SCAN COL3
	GPIOB->ODR |=(1<<11);	//COLOCA PB11 COMO 1
	GPIOC->ODR |=(1<<7);	//COLOCA PC7 COMO 1
	GPIOA->ODR |=(1<<9);	//COLOCA PA9 COMO 1
	GPIOC->ODR &=~(1<<4);	//COLOCA PC4 COMO 0
	delayMs(5);
	if(!(GPIOC->IDR & 0x20)){			//EVALUANDO FILA 0
		//SE PRESIONO LA TECLA A
		print_bcd_7_segment_decoder_CC(0x0A);
		GPIOA->ODR |=(1<<1);
		delayMs(5);
		//GPIOA->BSRR = (0x03<<16);
	}else if(!(GPIOC->IDR & 0x40)){		//EVALUANDO FILA 1
		//SE PRESIONO LA TECLA B
		print_bcd_7_segment_decoder_CC(0x0B);
		GPIOA->ODR |=(1<<1);
		delayMs(5);
		//GPIOA->BSRR = (0x03<<16);
	}else if(!(GPIOC->IDR & 0x100)){	//EVALUANDO FILA 2
		//SE PRESIONO LA TECLA C
		print_bcd_7_segment_decoder_CC(0x0C);
		GPIOA->ODR |=(1<<1);
		delayMs(5);
		//GPIOA->BSRR = (0x03<<16);
	}else if(!(GPIOC->IDR & 0x200)){	//EVALUANDO FILA 3
		//SE PRESIONO LA TECLA D
		print_bcd_7_segment_decoder_CC(0x0D);
		GPIOA->ODR |=(1<<1);
		delayMs(5);
		//GPIOA->BSRR = (0x03<<16);
	}
}
void print_bcd_7_segment_decoder_CC(uint8_t val){
last_displayed_value = val; ///////////////////////// // Actualiza el último valor mostrado

	if(val==0x00){						//ENCIENDE EL NUMERO 0 EN EL DISPLAY
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<8;	//SEGMENTO A
		GPIOB->ODR |=0x01<<2;	//SEGMENTO B
		GPIOB->ODR |=0x01<<3;	//SEGMENTO C
		GPIOB->ODR |=0x01<<4;	//SEGMENTO D
		GPIOB->ODR |=0x01<<6;	//SEGMENTO E
		GPIOB->ODR |=0x01<<7;	//SEGMENTO F
	}else if(val==0x01){				//ENCIENDE EL NUMERO 1 EN EL DISPLAY
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<2;	//SEGMENTO B
		GPIOB->ODR |=0x01<<3;	//SEGMENTO C
	}else if(val==0x02){				//ENCIENDE EL NUMERO 2 EN EL DISPLAY
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<8;	//SEGMENTO A
		GPIOB->ODR |=0x01<<2;	//SEGMENTO B
		GPIOB->ODR |=0x01<<9;	//SEGMENTO G
		GPIOB->ODR |=0x01<<6;	//SEGMENTO E
		GPIOB->ODR |=0x01<<4;	//SEGMENTO D
	}else if(val==0x03){				//ENCIENDE EL NUMERO 3 EN EL DISPLAY
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<8;	//SEGMENTO A
		GPIOB->ODR |=0x01<<2;	//SEGMENTO B
		GPIOB->ODR |=0x01<<9;	//SEGMENTO G
		GPIOB->ODR |=0x01<<3;	//SEGMENTO C
		GPIOB->ODR |=0x01<<4;	//SEGMENTO D
	}else if(val==0x04){				//ENCIENDE EL NUMERO 4 EN EL DISPLAY
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<7;	//SEGMENTO F
		GPIOB->ODR |=0x01<<9;	//SEGMENTO G
		GPIOB->ODR |=0x01<<2;	//SEGMENTO B
		GPIOB->ODR |=0x01<<3;	//SEGMENTO C
	}else if(val==0x05){				//ENCIENDE EL NUEMRO 5 EN EL DISPLAY
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<8;	//SEGMENTO A
		GPIOB->ODR |=0x01<<7;	//SEGMENTO F
		GPIOB->ODR |=0x01<<9;	//SEGMENTO G
		GPIOB->ODR |=0x01<<3;	//SEGMENTO C
		GPIOB->ODR |=0x01<<4;	//SEGMENTO D
	}else if(val==0x06){				//ENCIENDE EL NUEMRO 6 EN EL DISPLAY
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<7;	//SEGMENTO F
		GPIOB->ODR |=0x01<<6;	//SEGMENTO E
		GPIOB->ODR |=0x01<<4;	//SEGMENTO D
		GPIOB->ODR |=0x01<<3;	//SEGMENTO C
		GPIOB->ODR |=0x01<<9;	//SEGMENTO G
	}else if(val==0x07){				//ENCIENDE EL NUEMRO 7 EN EL DISPLAY
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<8;	//SEGMENTO A
		GPIOB->ODR |=0x01<<2;	//SEGMENTO B
		GPIOB->ODR |=0x01<<3;	//SEGMENTO C
	}else if(val==0x08){				//ENCIENDE EL NUEMRO 8 EN EL DISPLAY
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<8;	//SEGMENTO A
		GPIOB->ODR |=0x01<<2;	//SEGMENTO B
		GPIOB->ODR |=0x01<<3;	//SEGMENTO C
		GPIOB->ODR |=0x01<<4;	//SEGMENTO D
		GPIOB->ODR |=0x01<<6;	//SEGMENTO E
		GPIOB->ODR |=0x01<<7;	//SEGMENTO F
		GPIOB->ODR |=0x01<<9;	//SEGMENTO G
	}else if(val==0x09){				//ENCIENDE EL NUMERO 9 EN EL DISPLAY
		GPIOB->BSRR=(0x3DC<<16);	//RESETA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<9;	//SEGMENTO G
		GPIOB->ODR |=0x01<<7;	//SEGMENTO F
		GPIOB->ODR |=0x01<<8;	//SEGMENTO A
		GPIOB->ODR |=0x01<<2;	//SEGMENTO B
		GPIOB->ODR |=0x01<<3;	//SEGMENTO C
	}else if(val==0x0A){				//ENCIENDE LA LETRA A
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<8;	//SEGMENTO A
		GPIOB->ODR |=0x01<<2;	//SEGMENTO B
		GPIOB->ODR |=0x01<<3;	//SEGMENTO C
		GPIOB->ODR |=0x01<<6;	//SEGMENTO E
		GPIOB->ODR |=0x01<<7;	//SEGMENTO F
		GPIOB->ODR |=0x01<<9;	//SEGMENTO G
	}else if(val==0x0B){				//ENCIENDE LA LETRA b
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<3;	//SEGMENTO C
		GPIOB->ODR |=0x01<<4;	//SEGMENTO D
		GPIOB->ODR |=0x01<<6;	//SEGMENTO E
		GPIOB->ODR |=0x01<<7;	//SEGMENTO F
		GPIOB->ODR |=0x01<<9;	//SEGMENTO G
	}else if(val==0x0C){				//ENCIENDE LA LETRA C
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<8;	//SEGMENTO A
		GPIOB->ODR |=0x01<<4;	//SEGMENTO D
		GPIOB->ODR |=0x01<<6;	//SEGMENTO E
		GPIOB->ODR |=0x01<<7;	//SEGMENTO F
	}else if(val==0x0D){				//ENCIENDE LA LETRA d
		GPIOB->BSRR=(0x3DC<<16);	//RESETEA TODOS LOS BITS A 0
		GPIOB->ODR |=0x01<<2;	//SEGMENTO B
		GPIOB->ODR |=0x01<<3;	//SEGMENTO C
		GPIOB->ODR |=0x01<<4;	//SEGMENTO D
		GPIOB->ODR |=0x01<<6;	//SEGMENTO E
		GPIOB->ODR |=0x01<<9;	//SEGMENTO G
	}
}

void delayMs(int n){		//DELAY EXPERIMENTAL
	int i;
	for(; n > 0; n--)
		for(i = 0; i < 240; i++);
}

