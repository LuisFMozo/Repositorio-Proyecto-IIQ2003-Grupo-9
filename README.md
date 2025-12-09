README Proyecto IIQ2003 Grupo 9
El proyecto consiste en el modelamiento matemático - computacional de la evolución de temperatura dentro de un silo agrícola de almacenamiento de grano de maíz, considerando principalmente las fuentes externas e interiores de pérdida o generación de calor. La importancia de hacer esto se debe a la pérdida de calidad y deterioro del grano debido al desarrollo de hongos, insectos y microorganismos, por  un aumento de temperatura y humedad dentro de estos almacenes.
Esta problemática se hace especialmente importante en el contexto chileno, puesto que la agricultura representa un pilar importantísimo en la economía nacional y debido a los factores mencionados se puede llegar a perder hasta el 18% de la cosecha debido a este mal almacenamiento. El tener conocimiento sobre cómo se comporta la temperatura dentro de los silos, permitirá desarrollar técnicas de optimización permitiendo seguridad alimentaria, reducción del desperdicio de alimentos y un uso más eficiente de recursos como agua, energía y superficie agrícola. Lo cual se relaciona directamente con desafíos actuales de sustentabilidad no tan solo en Chile, sino que a nivel mundial, el tener herramientas analíticas de este tipo permite un almacenamiento más seguro, eficiente, el cual repercute directamente en un menor impacto ambiental.
El sistema físico empleado para lograr el modelamiento, fue de silo cilíndrico vertical, completamente cerrado, el cual contiene grano a granel en su interior. Si bien el sistema contenía fenómenos bastante relevantes tanto en el eje r como en el z, se decidió realizar un análisis en esta última coordenada, ya que en esta se presentan los mecanismos predominantes, tales como la convección natural generación interna de calor por respiración del grano y gradientes de temperatura verticales, además para una mayor representación de la realidad se utilizó un modelo transientes, pues estos silos están sometidos constantemente a cambios de temperatura, debido a los ciclos dia - noche, siendo de suma importancia incluir estos cambios en el análisis.
Los componentes principales de este silo son:
  - Masa de grano: medio poroso donde ocurre generación de calor y transferencia térmica.
  - Aire intergranular: participa en el transporte de calor por convección interna.
  - Temperatura interior y exterior.
  - Superficie superior e inferior: donde se imponen condiciones de contorno térmicas.
El principal fenómeno de transporte involucrado en nuestra modelo corresponde a la transferencia de calor por conducción y convección interna, sumado a efectos de generación metabólica del grano y pérdidas térmicas hacia el ambiente.
Los principales supuestos utilizados para este sistema fueron:
  1. Estado transiente.
  2. Aire tratado como fluido incompresible y newtoniano.
  3. Disipación viscosa despreciable.
  4. Flujo interno representativo mayoritariamente en el eje z.
  5. Silo cerrado y sin intercambio de masa con el exterior.
  6. Paredes con pérdida térmica por radiación y convección.
  7. El silo comienza la simulación “ya calentado”, sin influencia del proceso de llenado.
  8. Proporción radio/altura igual a 1/5
<img width="306" height="378" alt="Silo modelado" src="https://github.com/user-attachments/assets/a9f75d3c-e5dd-41c6-ae63-2596bd4caa66" />

Todo lo anterior se representa en nuestra siguiente ecuación gobernante:
<img width="616" height="147" alt="Ecuación gobernante" src="https://github.com/user-attachments/assets/a1d51c03-7656-4626-8a06-ed73e2058d8d" />

Para resolver esta ecuación, se utilizó el método número FTCS (Forward Time Centered Space), este esquema nos permite aproximar derivadas especiales mediante una discretización centrada y la derivada temporal mediante un avance hacia adelante en el tiempo, lo que nos permite realizar una actualización directa de la temperatura en cada nodo y paso temporal.
Nuestro sistema físico corresponde a un proceso transitorio, donde la temperatura del grano cambia según el tiempo debido a una conducción térmica, convección interna, generación metabólica y pérdidas térmicas, estos se discretizan reemplazando las derivadas por sus equivalentes en diferencias finitas. 
El dominio espacial (z) se divide en Nznodos uniformemente espaciados donde Delta_z= L/(Nz-1), luego el tiempo se discretiza en pasos constantes t, de esta forma la temperatura en un nodo espacial i y tiempo j queda de la forma T_i,j.
Para cada derivada parcial presente en nuestra ecuación gobernante se reemplazaron sus aproximaciones en diferencias finitas:

  - Derivada temporal (Aproximación hacia adelante de primer orden): <img width="110" height="40" alt="Derivada temporal" src="https://github.com/user-attachments/assets/6f7e04c9-7d29-494a-9da1-705bebb59301" />

  - Derivada espacial de convección (Aproximación central de segundo orden): <img width="120" height="45" alt="Derivada espacial de convección" src="https://github.com/user-attachments/assets/09a024e0-d1b9-4f70-ba27-38d59f49419e" />

  - Derivada espacial de conducción (Aproximación central de segundo orden): <img width="134" height="55" alt="Derivada espacial de conducción" src="https://github.com/user-attachments/assets/59c97d90-adb0-41e8-821f-eba714a29563" />
  
Reemplazando estos valores en nuestra ecuación gobernante se obtiene la ecuación: 

<img width="826" height="72" alt="Ecuación gobernante actualizada" src="https://github.com/user-attachments/assets/8b25dfc0-7acf-4e3f-b65c-aa2ca4bc0ea4" />

Luego de realizar un desarrollo algebraico se llega a una expresión para T_i,j+1:

<img width="831" height="87" alt="Temperatura en el tiempo periodo j+1 para el nodo i despejada" src="https://github.com/user-attachments/assets/ee1698b5-be71-43d6-86d6-fe40d9cd28ee" />

Expresión la cual es exactamente la forma matricial de la ecuación, de la forma <img width="140" height="40" alt="" src="https://github.com/user-attachments/assets/9ec312e3-67cf-43f4-a078-0fcf1522f613" /> , permitiendo que dependa únicamente de los valores en el tiempo previo y los parámetros físicos del sistema. Esto a su vez nos permite definir una matriz A y vector b, los cuales se rellenarán según nuestra discretización.
Una vez hecho todo lo anterior, para encontrar la evolución temporal de la temperatura se hace uso de un proceso iterativo, típico del método FTCS. Para lograr esto se hace uso de un ciclo ‘while’ en donde durante cada iteración se actualiza nuestra ecuación <img width="275" height="35" alt="" src="https://github.com/user-attachments/assets/be923d51-71f9-49cd-8934-37531a3f15e7" /> , esto se puede evidenciar en el código como ‘T_new = T_old + dt * (np.dot(A, T_old) + b)’, la nos permite actualizar la temperatura de cada nodo utilizando únicamente  los valores conocidos del tiempo anterior. 
Posteriormente es de suma importancia aplicar las condiciones de borde a nuestro T_new, esto se puede evidenciar en las líneas de códigos que parte por T_new [-1] y T_new[0], siendo las condiciones de frontera para z = L y z = 0. 
Tras corregir estos nodos y avanzar el tiempo, se almacena nuestra temperatura con el comando T_old = np.copy(T_new), lo que permite guardar la temperatura T_old como la actual, para utilizarla en la siguiente iteración, este paso es de suma importancia, ya que sin esta asignación, el método no podría continuar la iteración.
Finalmente con el bloque que se encuentra dentro del if se guardan los perfiles a intervalos de tiempo predeterminados, este paso si bien no afecta al cálculo, es de suma importancia para graficar la evolución de la temperatura.

En resumen el algoritmo a llevar a cabo es:
  1. Definir los parámetros físicos y geométricos
  2. Definir la retícula espacial 
  3. Construir nuestra matriz A y vector b según discretización 
  4. Aplicar las condiciones de borde a la matriz 
  5. Avanzar en el tiempo haciendo uso de Tj+1=Tj+t (ATj+b)
  6. Corregir nodos de frontera haciendo uso de las condiciones de borde
  7. Almacenar los resultados según tiempos seleccionados

Uno de los requisitos para usar este modelo es que la ecuación que tenemos sea de forma parabólica, donde <img width="213" height="37" alt="" src="https://github.com/user-attachments/assets/adb78f18-fbb6-4800-8e4d-9b34588d4304" /> Huerta Pérez (s.f.), requisito que en nuestro caso si se cumple. Nuestra ecuación se puede representar de la siguiente manera <img width="401" height="65" alt="" src="https://github.com/user-attachments/assets/e1c0efbb-3e5f-4427-8fb4-b144455ca1de" /> , para esta ecuación debemos buscar los coeficientes a, b y c que nos permitan clasificarla, para esto debemos buscar hacemos uso de la expresión general <img width="385" height="74" alt="" src="https://github.com/user-attachments/assets/4f42c7e2-210d-472e-8332-42396cf5a0ab" /> Huerta Pérez (s.f.), luego si nos fijamos a = k, b = 0, c = 0, por lo que <img width="251" height="47" alt="" src="https://github.com/user-attachments/assets/f98f5dbd-f648-4dfc-ac0d-9699ec80e8f1" /> , cumpliendo que se tiene una ecuación del tipo parabólica y se puede hacer uso del método FTCS.
Sumado a lo anterior el uso de este método es adecuado debido a varios puntos:
  - Simulación de procesos transitorios: El método permite simular procesos transientes, lo que en este caso es esencial para representar ciclos día/noche, generación metabólica dependiente de la temperatura, cambios graduales en la temperatura interna del grano.
  - Facilidad de implementación: El funcionamiento de FTCS se basa en el uso de valores del tiempo anterior, lo que permite que sea fácil de implementar, reduciendo el costo computacional. 
  - Flexibilidad para incluir términos: El método permite incluir términos adicionales sumamente importantes en nuestro caso, tales como generación metabólica dependiente de T, pérdidas térmicas radiativas (T4), convección interna. 

Todo lo anterior permite asegurar que el método empleado es correcto para un sistema físico 1D - transiente  donde la transferencia dominante ocurre a lo largo de un eje, permitiendo tener un balance entre precisión y simplicidad computacional.

Instrucciones ejecución del código:
Primero se deben importar los módulos numpy para el cálculo numérico y computación científica, matplotlib.pyplot para la presentación de gráficos y cm desde el módulo matplotlib para mapas de colores. Definimos nuestras condiciones iniciales de temperatura para el maíz a lo largo del silo y el suelo del silo. Luego definimos los parámetros físicos que se usarán en las ecuaciones diferenciales, junto con la función de temperatura del aire dependiente del tiempo. Estos datos los usaremos para todas las variaciones de tamaño de silos. 
Para el primer silo a analizar definimos sus dimensiones para largo, radio y volumen. Procedemos a definir los parámetros de simulación, El número de nodos es Z será 101, con un espaciamiento de tiempo de 600 segundos. Se define el reticulado “z” y su espaciamiento “dz”. Se define la condición inicial que indica que el maíz va a estar a la misma temperatura para todos los nodos espaciales, usando un vector T lleno con T_0. El tiempo inicial será 0 y el tiempo final 259200 representando 3 días medidos en segundos.
Se crea la matriz A con ceros y luego se rellena con los coeficientes de la discretización según el nodo a la izquierda, nodo central y nodo a la derecha. 
Se aplican las condiciones de borde para Z=0 que mantiene constante la temperatura del nodo más cercano al suelo y la condición para Z=L que evita el cambio de la temperatura en el nodo superior.
Ahora hacemos las iteraciones y grabamos la información cada 600 segundos. Creamos una lista t_0 vacío que recolecta los tiempos, una copia del vector T llamado T_old y una lista T_num que recolecta las temperaturas según los nodos. Hacemos la ecuación de evolución de T desde el tiempo inicial al final. Dentro del ciclo agregamos la evolución del tiempo sumándole a t_0 el diferencial de tiempo (600 segundos), definimos b ya que este irá cambiando según avance t_0, se escribe T_new que representa una iteración. Debido al escenario que se está modelando, dentro del ciclo también se agrega el efecto de convección que ocurre en la tapa del silo, luego se define T_new para el último nodo, que añade el efecto de convección del aire en la tapa, actuando como condición de borde en Z=L y se define T_new para el nodo 0 igual a  T_suelo, siendo la condición de borde en Z=0. Se actualiza T_old como copia de T_new y se guarda el perfil de temperatura cada vez que el tiempo alcanza un múltiplo de 600 segundos, agregando a T_num la temperatura en el paso del tiempo y en t_vec el tiempo correspondiente.
Luego se convierte la lista de listas T_num a una matriz cuyas filas representan el paso del tiempo y la columnas representan el avance en z, siendo la columna 0 el suelo del silo y la columna 101 la tapa. 
Para el primer gráfico se definen los nodos que se quieren revisar en una lista “indices_interes_general” y otra lista “alturas_general” con los valores que estos nodos representan en metros. Graficamos usando mapa de colores “plasma” donde la menor altura tendrá el color más oscuro y la más alta el color más claro. Usamos un ciclo para generar el gráfico, donde “i” nos dice la posición de altura (alturas_general) para tomar los distintos colores en el gráfico junto con su leyenda correspondiente, “j” es el valor de los nodos o índices que queremos ver (indices_interes_general). Con ax.plot dentro del ciclo se va generando el gráfico temperatura versus tiempo que contiene todas las curvas para los Z deseados, donde t_vec va a corresponder al eje “x” del gráfico representando el tiempo y usamos T_matriz[:, j] que va a ir tomando una columna y todas sus filas, representando todos los cambios de temperatura en el tiempo para el nodo j, estos valores van a estar en el eje “y” del gráfico. El ciclo se va a repetir el mismo número de veces que el número de elementos de la lista de alturas. Se agregan las etiquetas para cada eje y título en su formato. Para el segundo gráfico se grafican solo los índices inferiores y para el tercer gráfico solo los índices superiores usando indices_inferior, alturas_inferior, indices_superior y alturas_superior segun corresponda (reemplazando indices_interes_general y alturas_general).
Este procedimiento se repite dos veces más, alterando las dimensiones del silo, usando índices y alturas proporcionales a los nuevos largos para su graficación.

Este código se debe correr desde arriba hacia abajo para que entregue los valores correctos.

Gráficos,
Se realizaron 3 configuraciones de silos:

  - Silo 1: 10 metros de largo y 1 metro de radio. 
<img width="700" height="554" alt="Gráfico de Temperatura respecto al tiempo para puntos especificos del silo" src="https://github.com/user-attachments/assets/4ed521e1-f861-4407-b099-ababecde6cfc" />
<img width="760" height="554" alt="Gráfico de Temperatura respecto al tiempo para puntos cercanos a la parte superior del silo" src="https://github.com/user-attachments/assets/ec7aecb7-49ff-47a1-8d2f-d73323d99cab" />
<img width="754" height="554" alt="Gráfico de Temperatura respecto al tiempo para puntos cercanos a la parte inferior del silo" src="https://github.com/user-attachments/assets/8c6156c7-3715-4488-bcc8-0c4d1f6bbc08" />

  - Silo 2: 10 metros de largo y 2 metros de radio. 
<img width="700" height="554" alt="Gráfico de temperatura respecto al tiempo para puntos específicos del silo" src="https://github.com/user-attachments/assets/a5a6e672-1ba0-4516-894e-795c1ca7be49" />
<img width="760" height="554" alt="Gráfico de temperatura respecto al tiempo para puntos cercanos a la parte superior del silo" src="https://github.com/user-attachments/assets/1b7244f1-d9a9-41f3-88de-6126ea7ae88b" />
<img width="755" height="554" alt="Gráfico de temperatura respecto al tiempo para puntos cercanos a la parte inferior del silo" src="https://github.com/user-attachments/assets/126f7a80-b780-485f-868e-9b62e15e9b3e" />

  - Silo 3: 5 metros de largo y 1 metro de radio.
<img width="700" height="554" alt="Gráfico de la temperatura respecto al tiempo para puntos específicos del silo" src="https://github.com/user-attachments/assets/1c7a9b09-33a1-42fc-904b-0ccd76744e2c" />
<img width="760" height="554" alt="Gráfico de la temperatura respecto al tiempo para puntos cercanos a la parte superior del silo" src="https://github.com/user-attachments/assets/b1df3e5b-0a35-4cac-b068-adcef5d8dcad" />
<img width="755" height="554" alt="Gráfico de la temperatura respecto al tiempo para puntos cercanos a la parte inferior del silo" src="https://github.com/user-attachments/assets/47c0c0b6-c4ab-4dec-9795-791a99c6d3c9" />

Estos son los 3 gráficos por configuración de silo, estos se pueden encontrar tanto en el código principal, los cuales aparecen una vez se reproduce el código, como en la carpeta "Gráficos de los resultados".
En estos se puede evidenciar la evolución de temperatura dentro del silo para distintas alturas para el primero se evidencia la temperatura en alturas específicas del silo (indicadas en la leyenda), el segundo indica las temperaturas cerca de la parte inferior del silo (z = 0) y el último gráfico las temperaturas cerca de la parte superior del silo (z = L). Esta configuración de 3 gráficos se repite para cada configuración de los 3 silos.
Como se puede evidenciar en las 3 configuraciones se puede presenciar tendencias de temperatura que responden al ciclo día - noche, aumentando estas en el día y disminuyendo en la noche, como es esperable.
Dependiendo de la configuración geométrica que se usa los perfiles de temperatura cambian en los 3 casos, en algunos más y en otro menos, lo que permite realizar análisis en profundidad del porqué ocurre esto.

