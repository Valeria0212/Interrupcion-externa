# Ejemplo interrupcion externa

En este documento se explicará paso por paso como realizar una interupcion externa en la targeta STM32L476, por medio de un ejercicio muy sencillo.

El ejercico consiste en poner como salida los pines 0, 1, 6 y 7 del puerto A (en estos se conectaran leds), y como entrada el pin 13 del puerto C (allí está conectado el pulsador de la tarjeta) y cada que se detecte una interrupcion externa, en este caso cada que se presione el pulsador, se activaran diferentes pines del puerto A.

Antes de indicar los pasos para hanilitar la interrupcion externa se deben configurar los pines, asi:
```
// Se habilitan los relojes perifericos del GPIOA y GPIOC
 	RCC->AHB2ENR = 0x00000005;

	// Configuracion puerto A
	GPIOA->MODER &= 0xABFFFFFF;// resetea valores del puerto A
	GPIOA->MODER &= 0xFFFF5005;// Pone los pines a0, a1, a6, a7 como salida

	// Configuracion puerto C
	GPIOC->MODER &= 0xFFFFFFFF;// resetea valores del puerto C
	GPIOC->MODER &= 0xF3FFFFFF;// Pone el pin C13 como entrada

	// Se habilita el reloj SYSCFG (necesario para la interrupcion)
	RCC->APB2ENR = 0x00000001; //registro de habilitación de reloj periférico
```
