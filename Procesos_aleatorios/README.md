#           Procesos aleatorios
----------
Modelos Probabilísticos de Señales y Sistemas

Universidad de Costa Rica

Belinda Brown, B61254

Julio, 2020

----------

## Importante
Es necesario considerar que los resultados ajuntos son los pertencientes a la muestra 1, lo cual se entenderá cuando se analiza
la sección de decodificación y demodulación pero esta explicación viene dada en la modulación dado que es un efecto que se acarrea.


## Correr el programa
Con el fin de probar el programa, se requieren revisar los paths contenidos en el main.py el cual se encuentra dentro de la
carpeta de "src" ya que estos paths están dirigidos a mi máquina. Después de realizar esto se cuenta con un makefile el cual
permite probar el algoritmo y limpiar la carpeta de "results" en donde se encuentran los resultados. Se siguen los siguientes comandos.

Para analizar todo lo requerido se debe ingresar:

**1. <path_donde_se_encuentra_el_folder>$ make analize**
 
Para borrar los resultados obtenidos se debe digitar:

**2. <path_donde_se_encuentra_el_folder>$ make clean**

## Paquetes importados
~~~~
#******************************************************
#               IMPORTANDO PAQUETES
#******************************************************
import csv
import pandas as pd
import matplotlib.pyplot as plt
from numpy import *
from numpy import genfromtxt
import numpy as np
from scipy import signal
from scipy import integrate
import scipy.stats as stats
from matplotlib import cm
~~~~

## Definiciones 

~~~~

#******************************************************
#           DEFINICIONES
#******************************************************
def lectura_archivo_csv(path):
	with open(path, "r") as csv_file:
		# Leyendo cada celda y separandola con coma para poder interpretar los datos
		csv_reader = csv.reader(csv_file, delimiter=',')
		# Arerglo para guardar los datos (bits)
		data = []
		# Recorre todas las filas dentro del archivo cvs
		for filas_completas_data in csv_reader:
			# Se guardan los datos en un arreglo
			data.append(filas_completas_data[0])
	return data

def grap_onda_p(puntos_por_periodo, forma_onda):
	plt.plot(puntos_por_periodo, forma_onda)
	plt.xlabel('Tiempo [s]')
	plt.ylabel('Amplitud de la señal')
	plt.title('Onda portadora')
	plt.ticklabel_format(axis = "x", style = "sci", scilimits=(0,0))
	plt.savefig('/Users/belindabrown/Desktop/Procesos_aleatorios/results/onda_portadora.png')
	# #                   Limpia el area de graficacion
	plt.cla()
	return

def graph_perio_modulados(num_perio, puntos_senal, senal_modulada, periodo):
	puntos_periodo = np.linspace(0, num_perio*periodo, num_perio*puntos_senal)
	plt.figure()
	plt.plot(senal_modulada[0:num_perio*puntos_senal])
	plt.xlabel('Tiempo [s]')
	plt.ylabel('Amplitud de la señal')
	plt.title('Señal modulada con pb de ' +  str(num_perio) )
	plt.savefig('/Users/belindabrown/Desktop/Procesos_aleatorios/results/senal_modulada.png')
	# #                   Limpia el area de graficacion
	plt.cla()
	return

def promedio_potencia(periodo, cantidad_de_bits, linea_tx, senal_tx):
	Potencia_instantanea = senal_tx**2
	potencia_prom= integrate.trapz(Potencia_instantanea, linea_tx)/(cantidad_de_bits*periodo)
	return potencia_prom

def AWGN(snr_ingresada_dB, senal_modulada_tx, periodos, puntos_senal, P_senal):
	# El SNR es la proporción de la señal a ruido
	# (Signal to Noise) en dB.
	SNR = snr_ingresada_dB
	# Potencia del ruido para una proporcion de señal
	# a ruido
	P_ruido = P_senal / (10**(SNR / 10))
	# Desviación estándar del ruido
	sigma = np.sqrt(P_ruido)
	# Ruido --> noise(Pn = sigma^2)
	ruido_creado = np.random.normal(0, sigma, senal_modulada_tx.shape)
	# Simulando "el canal": señal recibida
	Rx = senal_modulada_tx + ruido_creado
	# Visualización de los primeros bits recibidos
	plt.figure()
	plt.plot(Rx[0:periodos*puntos_senal])
	plt.xlabel('Tiempo [s]')
	plt.ylabel('Amplitud de la señal')
	plt.title('AWGN de '+ str(snr_ingresada_dB)+ ' dB \ncon pb de ' +  str(periodos))
	plt.savefig('/Users/belindabrown/Desktop/Procesos_aleatorios/results/AWGN_%s.png'%snr_ingresada_dB)
	return Rx

def welch_senal(frecuencia_muestreo, senal_analizar, nombre, can_nperseg):
	fw, PSD = signal.welch(senal_analizar, frecuencia_muestreo, nperseg=can_nperseg)
	plt.figure()
	plt.semilogy(fw, PSD)
	plt.xlabel('Frecuencia [Hz]')
	plt.ylabel('Densidad espectral de potencia / V**2 [Hz]')
	plt.title('Densidad espectral de potencia '+nombre +' dB \ndel canal ruidoso con el método de Welch')
	plt.savefig('/Users/belindabrown/Desktop/Procesos_aleatorios/results/densidad_espectral_welch_%s.png'%nombre)
	return

def demodulacion_y_decodifcacion(cantidad_de_bits_dd, onda_portadora_dd, puntos_onda_port_dd, RX_x_dd, path, name):
	with open(path) as data:
	  csv_data_reader = csv.reader(data)
	  bits_csv_dd = [int(x[0]) for x in csv_data_reader]
	# Pseudo-energía  es la suma de la los valores
	Pseudo_energia_dd = np.sum(onda_portadora_dd**2)
	# Inicialización del vector de bits recibidos (recordando que Rx es la capa recepctora)
	bitsRx_dd = np.zeros(np.array(bits_csv_dd).shape)
	# Decodificación de la señal por detección de energía
	for k_dd, b_dd in enumerate(bits_csv_dd):
		Pseudo_energia_ond_por_dd = np.sum(RX_x_dd[k_dd*puntos_onda_port_dd:(k_dd+1)*puntos_onda_port_dd] * onda_portadora_dd)
		if Pseudo_energia_ond_por_dd > Pseudo_energia_dd/2:
			bitsRx_dd[k_dd] = 1
		else:
			bitsRx_dd[k_dd] = 0
	cantidad_errores_dd = np.sum(np.abs(bits_csv_dd - bitsRx_dd))
	BER_dd= cantidad_errores_dd/cantidad_de_bits_dd
	print("\nSe consideran {} errores en una cantidad total de {} bits para una tasa de error de {} en ".format(int(cantidad_errores_dd), int(cantidad_de_bits_dd), float(BER_dd)) + name)
	return BER_dd

def graph_valores_dos_funciones(valores_func0, nombre0, valores_func1, nombre1):
	guardo = nombre0+nombre1
	plt.figure()
	plt.scatter(valores_func1, valores_func0,cmap=cm.coolwarm)
	plt.xlabel(nombre1)
	plt.ylabel(nombre0 + "(dB)")
	plt.title(nombre0 +" vs " + nombre1)
	plt.savefig('/Users/belindabrown/Desktop/Procesos_aleatorios/results/%s.png'%guardo)
	return
~~~~

## Obteniendo los datos del csv

Se hace un llamado a la definición creada. Observar que se posee la ruta (path) de mi máquina por si se desea probar se debe modificar esta ruta.
~~~~
#******************************************************
#          OBTENIENDO VALORES
#		   DE LOS CSV
#******************************************************
bits = lectura_archivo_csv("/Users/belindabrown/Desktop/Procesos_aleatorios/data_base/bits10k.csv")
~~~~

## Parametrización de ciertas características

Se realizó una sección en donde se parametrizaron ciertas características del modo en el que se pretende operar.

<img src="https://render.githubusercontent.com/render/math?math=f = 5000 Hz">

<img src="https://render.githubusercontent.com/render/math?math=T =  \frac{1}{f} ms">

<img src="https://render.githubusercontent.com/render/math?math=puntos = 50">

~~~~
#******************************************************
#         PARAMETRIZANDO CIERTAS
#	      CARACTERISTICAS DE OPERACION
#******************************************************
N_bits = len(bits) # Cantidd de bits
print("\nLa cantidad de bits a transmitir es de:          ", N_bits)
f = 5000 # Frecuencia de operacion [Hz]
T = 1/f # Periodo de onda --> inverso de la frecuencia [ms]
puntos = 50 # Se consideran 50 puntos de muestreo por operacion
puntos_periodo = np.linspace(0, T, puntos) # Puntos de muestreo por cada periodo
~~~~

## Modulación BPSK

<img src="https://render.githubusercontent.com/render/math?math=s(t) = A*sin(2*\pi*f*t)">

<img src="https://render.githubusercontent.com/render/math?math=s_{0}(t) = -A*sin(2*\pi*f*t)">

<img src="https://render.githubusercontent.com/render/math?math=f_{muestro}= \frac{puntos}{f} ">

~~~~
#******************************************************
#       ESQUEMA DE MODULACION BPSK PARA LOS BITS
#       ------ ONDA PORTADORA -------
#******************************************************
# Se considera que la amplitud es de uno pero se coloca un parámetro
# por si se desea en el futuro modificarlo
A = 1 # Donde A es amplitud de la onda portadora
onda_p = A*(np.sin(2*np.pi * f * puntos_periodo))
grafica_onda_port = grap_onda_p(puntos_periodo, onda_p)
#*********************************************************
######  ######  Para la señal modulada  ######  ######
#*********************************************************
f_muestro = puntos/T # Frecuencia de muestreo ---> 50 kHz
# Línea temporal para toda la señal Tx (trasmision)
l_temp = np.linspace(0, N_bits*T, N_bits*puntos)
# Inicializar el vector de la señal modulada Tx (transmision)
senal_tx_modu = np.zeros(l_temp.shape)
# Creación de la señal modulada BPSK
for k, b in enumerate(bits):
  mapped_b = int(''.join(map(str,b)))
  if mapped_b != 0:
	  senal_tx_modu[k*puntos:(k+1)*puntos] = mapped_b*onda_p
  else:
	  senal_tx_modu[k*puntos:(k+1)*puntos] = -1*onda_p
molada_tx = graph_perio_modulados(15, puntos, senal_tx_modu, T)
~~~~
### Onda portadora 
![image](https://github.com/brown9804/Modelos_Probabilisticos/blob/master/Procesos_aleatorios/results/onda_portadora.png)

### Señal modulada
![image](https://github.com/brown9804/Modelos_Probabilisticos/blob/master/Procesos_aleatorios/results/senal_modulada.png)


## Promedio de potencia

<img src="https://render.githubusercontent.com/render/math?math=P(T) = \frac{1}{2T}\int_{-T}^{T}s^2(t)">

Donde dt es aproximadamente:

<img src="https://render.githubusercontent.com/render/math?math=dt = A*s^2(t)">

O bien, se conoce como:

<img src="https://render.githubusercontent.com/render/math?math=P(T)= \frac{1}{2T}\int_{-T}^{T}E[X^2(t)]">
	  
Donde dt es aproximadamente:

<img src="https://render.githubusercontent.com/render/math?math=dt = A*E[X^2(t)]">
	  
~~~~
#******************************************************
#      			PROMEDIO DE
#      			POTENCIA
#******************************************************
promedio_potencia = promedio_potencia(T, N_bits, l_temp, senal_tx_modu)
print("El promedio de potencia es de:			", promedio_potencia, " W\n\n")
~~~~

## Canal de ruido tipo AWGN

Se considera un rango de -2 dB a 3 dB por lo que se adjuntan la gráfica pertenecientes al canal de ruido para cada valor.
Es importante considerar que para el canal de ruido se utiliza se le ingresa un ruido creado por una función donde se 
aplican valores aleatorios ya que "np.random.normal(0, sigma, senal_modulada_tx.shape)" utiliza el paquete de aleatorio
"random" por lo que cada vez que se ejecute este programa los resultados serán similares pero no iguales 
y esto se acarrarea conforme se continua en el proceso de densidad espectral, decodificación y errores.

<img src="https://render.githubusercontent.com/render/math?math=SNR_{dB} = 10 \log_{10}\left(\frac{P_s}{P_n}\right) ">

Donde:

Pn es la potencia del ruido ya que n por su sigla en inglés se refiere a noise y Ps es la potencia de la señal.

~~~~
#******************************************************
#      	CANAL RUIDO TIPO AWGN
#      	CON RELACION SENAL DE RUIDO SNR -2 a 3dB
#******************************************************
# AWGN significa Additive white Gaussian noise dada su
# traducción al espanol ruido aditivo blanco
# gaussiano.
Rx_me2 = AWGN(-2, senal_tx_modu, 15, puntos, promedio_potencia)
Rx_me1 = AWGN(-1, senal_tx_modu, 15, puntos, promedio_potencia)
Rx_0 = AWGN(0, senal_tx_modu, 15, puntos, promedio_potencia)
Rx_1 = AWGN(1, senal_tx_modu, 15, puntos, promedio_potencia)
Rx_2 = AWGN(2, senal_tx_modu, 15, puntos, promedio_potencia)
Rx_3 = AWGN(3, senal_tx_modu, 15, puntos, promedio_potencia)
~~~~
### -2 dB
![image](https://github.com/brown9804/Modelos_Probabilisticos/blob/master/Procesos_aleatorios/results/AWGN_-2.png)

### -1 dB
![image](https://github.com/brown9804/Modelos_Probabilisticos/blob/master/Procesos_aleatorios/results/AWGN_-1.png)

### 0 dB
![image](https://github.com/brown9804/Modelos_Probabilisticos/blob/master/Procesos_aleatorios/results/AWGN_0.png)

### 1 dB
![image](https://github.com/brown9804/Modelos_Probabilisticos/blob/master/Procesos_aleatorios/results/AWGN_1.png)

### 2 dB
![image](https://github.com/brown9804/Modelos_Probabilisticos/blob/master/Procesos_aleatorios/results/AWGN_2.png)

### 3 dB
![image](https://github.com/brown9804/Modelos_Probabilisticos/blob/master/Procesos_aleatorios/results/AWGN_3.png)




## Densidad espectral de potencia mediante el método de Welch

~~~~
#******************************************************
#      	DENSIDAD ESPECTRAL DE POTENCIA
#       METODO DE WELCH ANTES Y DESPUES
# 	    DEL CANAL DE RUIDO
#******************************************************
welch_antes = welch_senal(f_muestro, senal_tx_modu, "antes", 1024)
#*********************************************************
######  ######  Para después del canal con ruido  ######  ######
#*********************************************************
welch_despues_me2 = welch_senal(f_muestro, Rx_me2, "despues_-2", 1024)
welch_despues_me1 = welch_senal(f_muestro, Rx_me1, "despues_-1", 1024)
welch_despues_0 = welch_senal(f_muestro, Rx_0, "despues_0", 1024)
welch_despues_1 = welch_senal(f_muestro, Rx_1, "despues_1", 1024)
welch_despues_2 = welch_senal(f_muestro, Rx_2, "despues_2", 1024)
welch_despues_3 = welch_senal(f_muestro, Rx_3, "despues_3", 1024)
~~~~
### Antes del ruido
![image](https://github.com/brown9804/Modelos_Probabilisticos/blob/master/Procesos_aleatorios/results/densidad_espectral_welch_antes.png)

### Después de agregarle el ruido
#### -2 dB
![image](https://github.com/brown9804/Modelos_Probabilisticos/blob/master/Procesos_aleatorios/results/densidad_espectral_welch_despues_-2.png)

#### -1 dB
![image](https://github.com/brown9804/Modelos_Probabilisticos/blob/master/Procesos_aleatorios/results/densidad_espectral_welch_despues_-1.png)

#### 0 dB
![image](https://github.com/brown9804/Modelos_Probabilisticos/blob/master/Procesos_aleatorios/results/densidad_espectral_welch_despues_0.png)

#### 1 dB
![image](https://github.com/brown9804/Modelos_Probabilisticos/blob/master/Procesos_aleatorios/results/densidad_espectral_welch_despues_1.png)

#### 2 dB
![image](https://github.com/brown9804/Modelos_Probabilisticos/blob/master/Procesos_aleatorios/results/densidad_espectral_welch_despues_2.png)

#### 3 dB
![image](https://github.com/brown9804/Modelos_Probabilisticos/blob/master/Procesos_aleatorios/results/densidad_espectral_welch_despues_3.png)






## Demodulación y decodificación
En esta sección se tiene que:

<img src="https://render.githubusercontent.com/render/math?math=\langle g(t), h(t) \rangle = \int_0^T g(t) h(t)dt ">

Donde se considera que: 

<img src="https://render.githubusercontent.com/render/math?math=\langle g(t), g(t) \rangle = E_{g}">  Siendo esto la energía interna de la señal


<img src="https://render.githubusercontent.com/render/math?math=\langle g(t), h(t) \rangle = 0"> Para el caso de ortogonalidad, es decir, si existe un ángulo de 
90 grados entre ellas.

~~~~
#******************************************************
#      	DEMODULACION Y DECODIFICACION DE LA SENAL,
#       CONTEO DE LA TASA DE ERROR DE BITS
# 	    BER (BIT ERROR RATE) PARA CADA SNR
#******************************************************
path = "/Users/belindabrown/Desktop/Procesos_aleatorios/data_base/bits10k.csv"
BER_me2 = demodulacion_y_decodifcacion(N_bits, onda_p, puntos, Rx_me2, path, "-2 dB")
BER_me1 = demodulacion_y_decodifcacion(N_bits, onda_p,  puntos, Rx_me1, path, "-1 dB")
BER_0 = demodulacion_y_decodifcacion(N_bits, onda_p, puntos, Rx_0, path, "0 dB")
BER_1 = demodulacion_y_decodifcacion(N_bits, onda_p,puntos, Rx_1, path, "1 dB")
BER_2 = demodulacion_y_decodifcacion(N_bits, onda_p,  puntos, Rx_2, path, "2 dB")
BER_3 = demodulacion_y_decodifcacion(N_bits, onda_p,puntos, Rx_3, path, "3 dB")
BER_me2a3 = [BER_me2, BER_me1, BER_0, BER_1, BER_2, BER_3]
SNR_x = list(range(-2,4))
~~~~

Así como se mencionó anteriormente, dado que cada vez que se ejecuta el programa los valores varían un poco se 
presenta la prueba de lo mismo ya que anteriormente se explicó el porqué sucede este fenómeno.

### Muestra 0
![image](https://github.com/brown9804/Modelos_Probabilisticos/blob/master/Procesos_aleatorios/results/result0.png)


### Muestra 1
![image](https://github.com/brown9804/Modelos_Probabilisticos/blob/master/Procesos_aleatorios/results/r1.png)



## Gráfica de BNR vs SNR
~~~~
#******************************************************
#      	BER vs SNR
# 		Rango -2 dB a 3 dB
#******************************************************

BER_vs_SNR = graph_valores_dos_funciones(BER_me2a3, "BER", SNR_x, "SNR")
~~~~

Se adjunta la gráfica de la muestra 1. Se puede apreciar que conforme se aumentan los decibles el BER disminuye, 
donde BER hace referencia a sus siglas Bit Error Rate.

### Muestra 1
![image](https://github.com/brown9804/Modelos_Probabilisticos/blob/master/Procesos_aleatorios/results/BERSNR.png)

