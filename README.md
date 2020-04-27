# Ejemplo interrupcion externa

En este documento se explicará paso por paso como realizar una interupcion externa en la targeta STM32L476, por medio de un ejercicio muy sencillo.

El ejercico consiste en poner como salida los pines 0, 1, 6 y 7 del puerto A (en estos se conectaran leds), y como entrada el pin 13 del puerto C (allí está conectado el pulsador de la tarjeta) y cada que se detecte una interrupcion externa, en este caso cada que se presione el pulsador, se activaran diferentes pines del puerto A.

Antes de indicar los pasos para habilitar la interrupcion externa se deben configurar los pines de los puertos A y C, asi: (esta parte va dentro del main)

### Habilitacion de los relojes
•Para empezar se habilitan los relojes periféricos del GPIOA (leds) y GPIOC (botón)

![relojes](https://github.com/Valeria0212/Interrupcion-externa/blob/master/Imagenes/relojes.jpg)

 0101 en hexadecimal es 5
```
       // Se habilitan los relojes perifericos del GPIOA y GPIOC
 	RCC->AHB2ENR = 0x00000005;
```

• Se configuran los pines de los puertos
```
	// Configuracion puerto A
	GPIOA->MODER &= 0xABFFFFFF;// resetea valores del puerto A
	GPIOA->MODER &= 0xFFFF5005;// Pone los pines a0, a1, a6, a7 como salida

	// Configuracion puerto C
	GPIOC->MODER &= 0xFFFFFFFF;// resetea valores del puerto C
	GPIOC->MODER &= 0xF3FFFFFF;// Pone el pin C13 como entrada

	// Se habilita el reloj SYSCFG (necesario para la interrupcion)
	RCC->APB2ENR = 0x00000001; //registro de habilitación de reloj periférico
```
Para entender mejor la configuracion de pines da click [aquí](https://github.com/MarianaEstrada/Guia_GPIO/blob/master/README.md).

## Descripcion del controlador extendido de interrupciones y eventos -EXTI 

Para las líneas de interrupción configurables, la línea de interrupción debe configurarse y habilitarse para generar una interrupción. Esto se hace programando los dos registros de disparo con la detección de borde deseada y habilitando la solicitud de interrupción escribiendo un "1" en el bit correspondiente en el registro de la máscara de interrupción. Cuando el borde seleccionado se produce en la línea de interrupción, se genera una solicitud de interrupción. También se establece el bit pendiente correspondiente a la línea de interrupción. Esta solicitud se borra escribiendo un "1" en el registro pendiente.


