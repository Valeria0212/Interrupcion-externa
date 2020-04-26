# Ejemplo interrupcion externa

En este documento se explicará paso por paso como realizar una interupcion externa en la targeta STM32L476, por medio de un ejercicio muy sencillo.

El ejercico consiste en poner como salida los pines 0, 1, 6 y 7 del puerto A (en estos se conectaran leds), y como entrada el pin 13 del puerto C (allí está conectado el pulsador de la tarjeta) y cada que se detecte una interrupcion externa, en este caso cada que se presione el pulsador, se activaran diferentes pines del puerto A.

Antes de 
