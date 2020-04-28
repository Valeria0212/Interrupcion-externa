# Ejemplo interrupcion externa

En este documento se explicará paso por paso como realizar una interupcion externa en la targeta STM32L476, por medio de un ejercicio muy sencillo.

El ejercico consiste en poner como salida los pines 0, 1, 6 y 7 del puerto A (en estos se conectaran leds), y como entrada el pin 13 del puerto C (allí está conectado el pulsador de la tarjeta) y cada que se detecte una interrupcion externa, en este caso cada que se presione el pulsador, se activaran diferentes pines del puerto A.

Antes de indicar los pasos para habilitar la interrupcion externa se deben configurar los pines de los puertos A y C, asi: (esta parte va dentro del main)


•Para empezar se habilitan los relojes periféricos del GPIOA (leds) y GPIOC (botón), como queremos habilitar estos puertos, usamos el registro AHB2ENR que es el que los habilita:

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
```
Para entender mejor la configuracion de pines da click [aquí](https://github.com/MarianaEstrada/Guia_GPIO/blob/master/README.md).

Ahora pasamos a la configuracion de la interrupcion externa, no sin antes hacer una brebe descripcion.

### Descripcion del controlador extendido de interrupciones y eventos -EXTI 

Para las líneas de interrupción configurables, la línea de interrupción debe configurarse y habilitarse para generar una interrupción. Esto se hace programando los dos registros de disparo con la detección de borde deseada y habilitando la solicitud de interrupción escribiendo un "1" en el bit correspondiente en el registro de la máscara de interrupción. Cuando el borde seleccionado se produce en la línea de interrupción, se genera una solicitud de interrupción. También se establece el bit pendiente correspondiente a la línea de interrupción. Esta solicitud se borra escribiendo un "1" en el registro pendiente.

### Habilitar y borrar interrupciones de excepción de FPU
Las banderas de excepción FPU están generando una interrupción a través del controlador de interrupción. La interrupción de la FPU se controla globalmente a través del controlador de interrupción.
También se proporciona un bit de máscara en el controlador de configuración del sistema (SYSCFG), que permite habilitar / deshabilitar individualmente cada generación de interrupción de bandera de FPU.

Para habilitar esta bit de mascara se hace uso del registro APB2ENR:

![relojint](https://github.com/Valeria0212/Interrupcion-externa/blob/master/Imagenes/relojint.jpg)

```
	// Se habilita el bit de mascara SYSCFG (necesario para la interrupcion)
	RCC->APB2ENR = 0x00000001; //registro de habilitación 
```
Y luego se vincula el pin 13 (el de la interrupcion) con el EXTI 
```
	// Escribiendo un 0b0010 en el pin 13, vincula PC13 con EXTI4
	SYSCFG->EXTICR[3] |= 0x0020;	// Write 0002 to map PC13 to EXTI4
```

Ahora, para configurar una línea como fuente de interrupción, se utiliza el siguiente procedimiento- tomado de [STM32](https://app.luminpdf.com/viewer/5ea76e0e1f147d0017bbb725), pagina 402:
1. Configure el bit de máscara correspondiente en el registro EXTI_IMR.
![IMR](https://github.com/Valeria0212/Interrupcion-externa/blob/master/Imagenes/IMR.jpg)

La IMR1 por que está el bit de interés (13)


2. Configure los bits de selección de disparador de la línea de interrupción (EXTI_RTSR y EXTI_FTSR).

![RTS](https://github.com/Valeria0212/Interrupcion-externa/blob/master/Imagenes/RTS.jpg)

La RTSR1 por que está el bit de interés (13)

3. Configure los bits de habilitación y máscara que controlan el canal NVIC IRQ asignado al EXTI para que una interrupción proveniente de una de las líneas EXTI pueda ser correctamente admitida.

El gestor de interrupciones es quien se encarga de lidiar con eventos asincrónicos. La mayoría de estos eventos vienen de periféricos de hardware. Por ejemplo, un temporizador que alcanza un valor de período configurado, o un puerto UART que advierte sobre la llegada de datos. Otros son originados por el “mundo exterior”, por ejemplo, el usuario presiona un interruptor que hace que se genere una interrupción.

Una vez seleccionado el pin que desea usar se debe manejar la interrupción de alguna manera. Este proceso se describe a continuación y para hacerlo debemos tener en cuenta la siguiente tabla

![IMR](https://github.com/Valeria0212/Interrupcion-externa/blob/master/Imagenes/IRQ.JPG)

El pin 13 (el del pulsador) esta conectado con el manejador para pines conectados a la línea 10 a 15, por esto se pone como prioridad este manejador de interrupcion y a su vez se habilita, asi:
```
	// Establecer prioridad para cada solicitud de interrupción.  pin 13 boton
	NVIC->IP[EXTI15_10_IRQn] = 0x10;	// Priority level 1
	// enable EXTI15_10 IRQ from NVIC
	NVIC_EnableIRQ(EXTI15_10_IRQn);

```


