#Código del Trabajo de Fin de Máster realizado en el lenguaje de programación Python.
#TFM Sergio Ortiz Pérez-Ojeda
#Universidad Internacional de Valencia

#Librerías necesarias
import pandas as pd
import statistics as stats
import seaborn as sns
import matplotlib.pyplot as plt 
import numpy as np

#Lectura ficheros de datos
Players = pd.read_excel('TFM.xlsx', sheet_name='Jugadores')
Stats = pd.read_excel('TFM.xlsx', sheet_name='Stats')

###PREPAREACION DE DATOS: Players

Players.head()
#Se disponen de 4550 registros y 8 variables en el fichero de jugadores
Players.shape

#Comprobamos el tipo de cada variable
Players.dtypes

#Cambiamos el tipo de algunas varibales 
Players['Peso'] = Players['Peso'].astype('Int64')
Players['Año_Nacimiento'] = Players['Año_Nacimiento'].astype('Int64')

#Ahora son enteros
Players.dtypes

#Comprobamos los valores nulos
Players.isnull().sum()
#Existen nulos en los siguientes campos numericos peso:6 nacimiento:31

#Creamos dataframe con los datos numéricos
PlayersNums = Players.iloc[:,[1,2,4,6]]

#Eliminamos los valores nulos para obtener las medias reales
PlayersNums_sin_nan = PlayersNums.dropna(how='any')
PesoMedio=stats.mean(PlayersNums_sin_nan.Peso)


#Rellenamos nulos con cero
PlayersNums.fillna(0, inplace=True)

#Ahora observamos la distribución de ambas variables, para ver como imputar los valores nulos
#Distribucion del peso
plt.hist(PlayersNums_sin_nan.Peso)
plt.title('Histograma de peso')
plt.xlabel('Valor')
plt.ylabel('Conteo')
plt.show()
#Mas de 2000 registros se encuentran en torno a las 200 libras,
#siendo el valor de la media 208 es correcto imputar la media a los valores nulos

#Creamos una variable llamada edad, donde se resta el año de inicio menos el de nacimiento
edad = np.zeros(len(PlayersNums_sin_nan))

for i in range(0,len(PlayersNums_sin_nan)):
    edad[i]=round((PlayersNums_sin_nan.iloc[i,0]-PlayersNums_sin_nan.iloc[i,3]),0)

#Distribucion de la edad
plt.hist(edad)
plt.title('Histograma de edad')
plt.xlabel('Valor')
plt.ylabel('Conteo')
plt.show()
#Mas de 3500 registros se encuentran entre 20 y 25, mientras que la media
#es de 24 por lo que un valor medio de resta al año de inicio de 24 es el correcto 
EdadMedia=round(stats.mean(edad),0)


#Imputamos la media a los nulos del Peso
for i in range(0,len(Players)):
    if PlayersNums.iloc[i,2]==0:
          PlayersNums.iloc[i,2]=PesoMedio
    else:
        PlayersNums.iloc[i,2]=PlayersNums.iloc[i,2]
#Comprobamos que el valor minimo ya no es 0, es 114
min(PlayersNums.Peso)


#Imputamos el año de comienzo - 25 a los nulos del Año nacimiento
for i in range(0,len(Players)):
    if PlayersNums.iloc[i,3]==0:
          PlayersNums.iloc[i,3]=PlayersNums.iloc[i,0]-24
    else:
        PlayersNums.iloc[i,3]=PlayersNums.iloc[i,3]
#Comprobamos que el valor minimo ya no es 0, es 1902
min(PlayersNums.Año_Nacimiento)

#Sustituimos las columnas en el conjunto de datos original
Players.Peso = PlayersNums.Peso
Players.Año_Nacimiento = PlayersNums.Año_Nacimiento

#Comprobamos valores minimos, ahora son correctos
min(Players.Peso)
min(Players.Año_Nacimiento)

#Separamos la columna Altura mediante el delimitador, y se añade
#al conjunto de datos
Altura = Players["Altura"].str.split('-', expand=True)
Altura.columns = ['Pies', 'Pulgadas']
Players = pd.concat([Players, Altura], axis=1)

#Convertimos las variables Peso y Altura

#La primera de libras a kilos, 1 libra es igual a 0.453592 kilos
conversorlibras=0.453592
for i in range(0,len(Players)):
          Players.iloc[i,4]=round(Players.iloc[i,4]*conversorlibras,0)
          
#Ahora el peso minimo no es 114 lb, son 52 kg
min(Players.Peso)

#La segunda de pies y pulgadas a cm, 1 pie igual a 30,48 cm y 1 pulgada 2,54
#El primer paso es pasar las variables a float
Players['Pies'] = Players['Pies'].astype('float64')
Players['Pulgadas'] = Players['Pulgadas'].astype('float64')

conversorpies=30.48
conversorpulgadas=2.54
for i in range(0,len(Players)):
          Players.iloc[i,7]=round(Players.iloc[i,7]*conversorpies,2)
          Players.iloc[i,8]=round(Players.iloc[i,8]*conversorpulgadas,2)

#Sumamos ambas variables
for i in range(0,len(Players)):
          Players.iloc[i,3]=round(Players.iloc[i,7]+Players.iloc[i,8],0)
          
#Ahora el peso minimo no es 5 pies y 3 pulgadas, son 160 cm
min(Players.Altura)
max(Players.Altura)

#Cambiamos el tipo a entero de nuevo
Players['Peso'] = Players['Peso'].astype('Int64')
Players['Altura'] = Players['Altura'].astype('Int64')

#Eliminamos las variables pies y pulgadas, ya no aportan información
Players = Players.drop(['Pies','Pulgadas'], axis=1)

#Contamos por número de jugador
Players['Jugador'].value_counts()
#Existen aproximadamente unos 50 nombres que corresponden a más de un jugador
lista=[]
for i in range(0,len(Players)):
    if Players.iloc[i,0] not in lista:
          lista.append(Players.iloc[i,0])
    else:
        Players.iloc[i,0]=Players.iloc[i,0]+'1'
        if Players.iloc[i,0] not in lista:
          lista.append(Players.iloc[i,0])
        else:
            Players.iloc[i,0]=Players.iloc[i,0]+'2'

#Comprobamos que el problema se ha solucionado
Players['Jugador'].value_counts()

#Observamos los tipo de las variables, son todos correctos
Players.dtypes

#Observamos algunos parámetros del conjunto de datos final con el que vamos a trabajar
Players.describe()

###PREPAREACION DE DATOS: Stats

Stats.head()
#Se disponen de 24691 registros y 51 variables en el fichero de estadísticas
Stats.shape

#Comprobamos el tipo de cada variable
Stats.dtypes

#Comprobamos los valores nulos
Stats.isnull().sum()

#Se eliminan las filas que tienen valor nulo en jugador
Stats = Stats.dropna(subset=["Jugador"])

#Se eliminan las siguientes variables al no aportar información
Stats = Stats.drop(['3PAr','FTr','ORB%','DRB%','TRB%',
                    'AST%','STL%','BLK%','TOV%','USG%',
                    'OWS','DWS','WS','WS/48','OBPM',
                    'DBPM','BPM','VORP'], axis=1)

#La mayoria de los nulos que aparecen corresponden a datos anteriores a 1980,
#ya que existian menores facilidades para las recogidas de los datos, por lo que
#se va a filtrar a partir del año 1980
Stats = Stats[Stats.Año > 1979]

#Comprobamos los valores nulos de nuevo
Stats.isnull().sum()
#Existen valores nulos en las variables de porcentaje cuando el numero de 
#lanzamientos es cero por lo que hay que convertir esos nulos en cero

#Se disponen de 18927 registros y 33 variables tras la reduccion de variables
Stats.shape



#Se puede dividir la bbdd en tres apartados para facilitar la comprensión

#Datos del jugador
Datos = Stats.iloc[:,0:10]

#Comprobamos el tipo de cada variable
Datos.dtypes
#Debemos convertir todas las variables numéricas a enteras a excepcion del PER
Datos['Año'] = Datos['Año'].astype('Int64')
Datos['Edad'] = Datos['Edad'].astype('Int64')
Datos['Partidos'] = Datos['Partidos'].astype('Int64')
Datos['Partidos_Titular'] = Datos['Partidos_Titular'].astype('Int64')
Datos['Minutos'] = Datos['Minutos'].astype('Int64')


#Comprobamos los valores nulos
Datos.isnull().sum()

#Eliminamos la variable Partidos_Titular al contar con una gran cantidad de nulos
Datos = Datos.drop(['Partidos_Titular'], axis=1)

#Las variable PER al representar un porcentaje pequeño
#Eliminamos los valores nulos para obtener las medias reales
Datos_sin_nan = Datos.dropna(how='any')
PerMedio=stats.mean(Datos_sin_nan.PER)

#Distribucion del PER
plt.hist(Datos_sin_nan.PER)
plt.title('Histograma de PER')
plt.xlabel('Valor')
plt.ylabel('Conteo')
plt.show()
#Cerca de 16000 registros entre 0-15, mientras que la media es 
#de 12.39, por lo que es correcto imputarla

#Rellenamos nulos con -100, ya que existen jugador con valores 0 en minutos y PER
Datos.fillna(-100, inplace=True)

#PER: Player Eficiency Raring
for i in range(0,len(Datos)):
    if Datos.iloc[i,8]==-100:
          Datos.iloc[i,8]=round(PerMedio,1)
    else:
        Datos.iloc[i,8]=Datos.iloc[i,8]
#Valor mínimo 
min(Datos.PER)

#Separamos la variable posicion por el delimitador
Posición = Datos["Posición"].str.split('-', expand=True)
Posición.columns = ['Pos', 'Pos2']
Datos = pd.concat([Datos, Posición], axis=1)

#Eliminamos la variable Posicion original, porque ya no es necesaria
Datos = Datos.drop(['Posición'], axis=1)

#Ahora se une este dataframe con Players que también disponian de datos del jugador
Final = pd.merge(Players, Datos, on='Jugador')

#Comprobamos el tipo de cada variable, son correctos
Final.dtypes 

#Comprobamos los valores nulos, solo nulos en Universidad y posicion_sec, es correcto
Final.isnull().sum()
#La unión se ha realizado correctamente 

#Renombramos los datos de las posiciones principal y secundaria

#Primero observamos el conteo por posicion
Final['Pos'].value_counts()
Final['Pos2'].value_counts()
#Base -> PG o G
#Escolta -> SG
#Alero -> SF o F
#Ala-Pivot -> PF
#Pivot -> C

#Posicion principal
for i in range(0,len(Final)):
    if Final.iloc[i,14]=='PG' or Final.iloc[i,14]=='G':
          Final.iloc[i,14]='Base'
    elif Final.iloc[i,14]=='SG':
          Final.iloc[i,14]='Escolta'
    elif Final.iloc[i,14]=='SF' or Final.iloc[i,14]=='F':
          Final.iloc[i,14]='Alero'
    elif Final.iloc[i,14]=='PF':
          Final.iloc[i,14]='Ala-Pivot'      
    else:
        Final.iloc[i,14]='Pivot'

#Posicion secundaria
for i in range(0,len(Final)):
    if Final.iloc[i,15]=='PG' or Final.iloc[i,15]=='G':
          Final.iloc[i,15]='Base'
    elif Final.iloc[i,15]=='SG':
          Final.iloc[i,15]='Escolta'
    elif Final.iloc[i,15]=='SF' or Final.iloc[i,15]=='F':
          Final.iloc[i,15]='Alero'
    elif Final.iloc[i,15]=='PF':
          Final.iloc[i,15]='Ala-Pivot'  
    elif Final.iloc[i,15]=='C':
          Final.iloc[i,15]='Pivot'         
    else:
        Final.iloc[i,15]=Final.iloc[i,15]
        
#Comprobamos los nuevos valores
Final['Pos'].value_counts()
Final['Pos2'].value_counts()


#Estadisticas en el tiro más la ultima columna de puntos
Tiros = Stats.iloc[:,10:24]
Tiros = pd.concat([Tiros, Stats.PTS], axis=1)

#Comprobamos el tipo de cada variable
Tiros.dtypes

#Convertimos las variables a enteras, a excepcion de los porcentajes
Tiros['FG'] = Tiros['FG'].astype('Int64') #Tiros de campo convertidos
Tiros['FGA'] = Tiros['FGA'].astype('Int64') #Tiros de campo intentados
Tiros['3P'] = Tiros['3P'].astype('Int64') #Tiros de tres convertidos
Tiros['3PA'] = Tiros['3PA'].astype('Int64') #Tiros de tres intentados
Tiros['2P'] = Tiros['2P'].astype('Int64') #Tiros de dos convertidos
Tiros['2PA'] = Tiros['2PA'].astype('Int64') #Tiros de dos intentados
Tiros['FT'] = Tiros['FT'].astype('Int64') #Tiros libres convertidos
Tiros['FTA'] = Tiros['FTA'].astype('Int64') #Tiros libres intentados
Tiros['PTS'] = Tiros['PTS'].astype('Int64') #Puntos por temporada

#Comprobamos los valores nulos
Tiros.isnull().sum()
#Unicamente se obtienen nulos en los procentajes, los rellenamos con 0
Tiros.fillna(0, inplace=True)

#Añadimos la variable jugador para hacer el cruce con Players
Tiros = pd.concat([Tiros, Stats.Jugador], axis=1)
Final1 = pd.merge(Players, Tiros, on='Jugador')
#Eliminamos variables redundantes
Final1 = Final1.iloc[:,7:25]

#Resto de estadisticas (asistencias,perdidas,rebotes...)
Generales = Stats.iloc[:,24:32]

#Comprobamos el tipo de cada variable
Generales.dtypes

#Convertimos todas las variables a enteras
Generales['ORB'] = Generales['ORB'].astype('Int64') #Rebotes Ofensivos
Generales['DRB'] = Generales['DRB'].astype('Int64') #Rebotes Defensivos
Generales['TRB'] = Generales['TRB'].astype('Int64') #Rebotes Totales
Generales['AST'] = Generales['AST'].astype('Int64') #Asistencias
Generales['STL'] = Generales['STL'].astype('Int64') #Robos
Generales['BLK'] = Generales['BLK'].astype('Int64') #Tapones
Generales['TOV'] = Generales['TOV'].astype('Int64') #Pérdidas
Generales['PF'] = Generales['PF'].astype('Int64') #Faltas Personales

#Comprobamos los valores nulos
Generales.isnull().sum()
#No se contabilizan valores nulos

#Añadimos la variable jugador para hacer el cruce con Players
Generales = pd.concat([Generales, Stats.Jugador], axis=1)
Final2 = pd.merge(Players, Generales, on='Jugador')
#Eliminamos variables redundantes
Final2 = Final2.iloc[:,7:15]

#Base de datos final: TFM
TFM = pd.concat([Final, Final1, Final2], axis=1)

#El conjunto de datos queda con 18260 registros y 39 variables
TFM.shape

#Renombramos las columnas que acabn en %, para no tener problemas
TFM.rename(columns={'TS%':'TSP','FG%':'FGP','3P%':'3PP','2P%':'2PP',
                    'eFG%':'eFGP','FT%':'FTP'},
               inplace=True)

#Existe un error de datos redundantes que se muestran con la variable TOT, son 
#registros totales que suman las stats cuando un jugador cambia de equipo en la misma 
#temporada, por ello se eliminan al no aportar información nueva
TFM = TFM[TFM.Equipo != 'TOT']

#Observamos el nuevo tamaño del conjunto de datos
TFM.shape
#16678 registros

#Reseteamos el indice para no tener problemas con los bucles
TFM=TFM.reset_index(drop=True)

#TRATMIENTO DE OUTLIERS: Variables de la matriz de correlacion
#En primer lugar, calculamos todos los limites inferiores y superiores,
#utilizando la fórmula del IQR. (LI=Q1-1.5xIQR, LS=Q3+1.5xIQR)
Q1_Altura , Q3_Altura = TFM.Altura.quantile([0.25,0.75])
IQR_Altura = Q3_Altura - Q1_Altura
LI_Altura = Q1_Altura - 1.5*IQR_Altura
LS_Altura = Q3_Altura + 1.5*IQR_Altura
Q1_Peso , Q3_Peso = TFM.Peso.quantile([0.25,0.75])
IQR_Peso = Q3_Peso - Q1_Peso
LI_Peso = Q1_Peso - 1.5*IQR_Peso
LS_Peso = Q3_Peso + 1.5*IQR_Peso
Q1_Edad , Q3_Edad = TFM.Edad.quantile([0.25,0.75])
IQR_Edad = Q3_Edad - Q1_Edad
LI_Edad = Q1_Edad - 1.5*IQR_Edad
LS_Edad = Q3_Edad + 1.5*IQR_Edad
Q1_Minutos , Q3_Minutos = TFM.Minutos.quantile([0.25,0.75])
IQR_Minutos = Q3_Minutos - Q1_Minutos
LI_Minutos = Q1_Minutos - 1.5*IQR_Minutos
LS_Minutos = Q3_Minutos + 1.5*IQR_Minutos
Q1_PER , Q3_PER = TFM.PER.quantile([0.25,0.75])
IQR_PER = Q3_PER - Q1_PER
LI_PER = Q1_PER - 1.5*IQR_PER
LS_PER = Q3_PER + 1.5*IQR_PER
Q1_TSP , Q3_TSP = TFM.TSP.quantile([0.25,0.75])
IQR_TSP = Q3_TSP - Q1_TSP
LI_TSP = Q1_TSP - 1.5*IQR_TSP
LS_TSP = Q3_TSP + 1.5*IQR_TSP
Q1_FG , Q3_FG = TFM.FG.quantile([0.25,0.75])
IQR_FG = Q3_FG - Q1_FG
LI_FG = Q1_FG - 1.5*IQR_FG
LS_FG = Q3_FG + 1.5*IQR_FG
Q1_FGA , Q3_FGA = TFM.FGA.quantile([0.25,0.75])
IQR_FGA = Q3_FGA - Q1_FGA
LI_FGA = Q1_FGA - 1.5*IQR_FGA
LS_FGA = Q3_FGA + 1.5*IQR_FGA
Q1_FGP , Q3_FGP = TFM.FGP.quantile([0.25,0.75])
IQR_FGP = Q3_FGP - Q1_FGP
LI_FGP = Q1_FGP - 1.5*IQR_FGP
LS_FGP = Q3_FGP + 1.5*IQR_FGP
Q1_3P , Q3_3P = TFM['3P'].quantile([0.25,0.75])
IQR_3P = Q3_3P - Q1_3P
LI_3P = Q1_3P - 1.5*IQR_3P
LS_3P = Q3_3P + 1.5*IQR_3P
Q1_3PA , Q3_3PA = TFM['3PA'].quantile([0.25,0.75])
IQR_3PA = Q3_3PA - Q1_3PA
LI_3PA = Q1_3PA - 1.5*IQR_3PA
LS_3PA = Q3_3PA + 1.5*IQR_3PA
Q1_3PP , Q3_3PP = TFM['3PP'].quantile([0.25,0.75])
IQR_3PP = Q3_3PP - Q1_3PP
LI_3PP = Q1_3PP - 1.5*IQR_3PP
LS_3PP = Q3_3PP + 1.5*IQR_3PP
Q1_2P , Q3_2P = TFM['2P'].quantile([0.25,0.75])
IQR_2P = Q3_2P - Q1_2P
LI_2P = Q1_2P - 1.5*IQR_2P
LS_2P = Q3_2P + 1.5*IQR_2P
Q1_2PA , Q3_2PA = TFM['2PA'].quantile([0.25,0.75])
IQR_2PA = Q3_2PA - Q1_2PA
LI_2PA = Q1_2PA - 1.5*IQR_2PA
LS_2PA = Q3_2PA + 1.5*IQR_2PA
Q1_2PP , Q3_2PP = TFM['2PP'].quantile([0.25,0.75])
IQR_2PP = Q3_2PP - Q1_2PP
LI_2PP = Q1_2PP - 1.5*IQR_2PP
LS_2PP = Q3_2PP + 1.5*IQR_2PP
Q1_eFGP , Q3_eFGP = TFM.eFGP.quantile([0.25,0.75])
IQR_eFGP = Q3_eFGP - Q1_eFGP
LI_eFGP = Q1_eFGP - 1.5*IQR_eFGP
LS_eFGP = Q3_eFGP + 1.5*IQR_eFGP
Q1_FT , Q3_FT = TFM.FT.quantile([0.25,0.75])
IQR_FT = Q3_FT - Q1_FT
LI_FT = Q1_FT - 1.5*IQR_FT
LS_FT = Q3_FT + 1.5*IQR_FT
Q1_FTA , Q3_FTA = TFM.FTA.quantile([0.25,0.75])
IQR_FTA = Q3_FTA - Q1_FTA
LI_FTA = Q1_FTA - 1.5*IQR_FTA
LS_FTA = Q3_FTA + 1.5*IQR_FTA
Q1_FTP , Q3_FTP = TFM.FTP.quantile([0.25,0.75])
IQR_FTP = Q3_FTP - Q1_FTP
LI_FTP = Q1_FTP - 1.5*IQR_FTP
LS_FTP = Q3_FTP + 1.5*IQR_FTP
Q1_PTS , Q3_PTS = TFM.PTS.quantile([0.25,0.75])
IQR_PTS = Q3_PTS - Q1_PTS
LI_PTS = Q1_PTS - 1.5*IQR_PTS
LS_PTS = Q3_PTS + 1.5*IQR_PTS
Q1_ORB , Q3_ORB = TFM.ORB.quantile([0.25,0.75])
IQR_ORB = Q3_ORB - Q1_ORB
LI_ORB = Q1_ORB - 1.5*IQR_ORB
LS_ORB = Q3_ORB + 1.5*IQR_ORB
Q1_DRB , Q3_DRB = TFM.DRB.quantile([0.25,0.75])
IQR_DRB = Q3_DRB - Q1_DRB
LI_DRB = Q1_DRB - 1.5*IQR_DRB
LS_DRB = Q3_DRB + 1.5*IQR_DRB
Q1_TRB , Q3_TRB = TFM.TRB.quantile([0.25,0.75])
IQR_TRB = Q3_TRB - Q1_TRB
LI_TRB = Q1_TRB - 1.5*IQR_TRB
LS_TRB = Q3_TRB + 1.5*IQR_TRB
Q1_AST , Q3_AST = TFM.AST.quantile([0.25,0.75])
IQR_AST = Q3_AST - Q1_AST
LI_AST = Q1_AST - 1.5*IQR_AST
LS_AST = Q3_AST + 1.5*IQR_AST
Q1_STL , Q3_STL = TFM.STL.quantile([0.25,0.75])
IQR_STL = Q3_STL - Q1_STL
LI_STL = Q1_STL - 1.5*IQR_STL
LS_STL = Q3_STL + 1.5*IQR_STL
Q1_BLK , Q3_BLK = TFM.BLK.quantile([0.25,0.75])
IQR_BLK = Q3_BLK - Q1_BLK
LI_BLK = Q1_BLK - 1.5*IQR_BLK
LS_BLK = Q3_BLK + 1.5*IQR_BLK
Q1_TOV , Q3_TOV = TFM.TOV.quantile([0.25,0.75])
IQR_TOV = Q3_TOV - Q1_TOV
LI_TOV = Q1_TOV - 1.5*IQR_TOV
LS_TOV = Q3_TOV + 1.5*IQR_TOV
Q1_PF , Q3_PF = TFM.PF.quantile([0.25,0.75])
IQR_PF = Q3_PF - Q1_PF
LI_PF = Q1_PF - 1.5*IQR_PF
LS_PF = Q3_PF + 1.5*IQR_PF

#Creamos un nuevo df sin datos outliers
TFMsin = TFM[(TFM.Altura < LS_Altura) & (TFM.Altura > LI_Altura)]
TFMsin = TFMsin[(TFMsin.Peso < LS_Peso) & (TFMsin.Peso > LI_Peso)]
TFMsin = TFMsin[(TFMsin.Edad < LS_Edad) & (TFMsin.Edad > LI_Edad)]
TFMsin = TFMsin[(TFMsin.Minutos < LS_Minutos) & (TFMsin.Minutos > LI_Minutos)]
TFMsin = TFMsin[(TFMsin.PER < LS_PER) & (TFMsin.PER > LI_PER)]
TFMsin = TFMsin[(TFMsin.TSP < LS_TSP) & (TFMsin.TSP > LI_TSP)]
TFMsin = TFMsin[(TFMsin.FG < LS_FG) & (TFMsin.FG > LI_FG)]
TFMsin = TFMsin[(TFMsin.FGA < LS_FGA) & (TFMsin.FGA > LI_FGA)]
TFMsin = TFMsin[(TFMsin.FGP < LS_FGP) & (TFMsin.FGP > LI_FGP)]
TFMsin = TFMsin[(TFMsin['3P'] < LS_3P) & (TFMsin['3P'] > LI_3P)]
TFMsin = TFMsin[(TFMsin['3PA'] < LS_3P) & (TFMsin['3PA'] > LI_3P)]
TFMsin = TFMsin[(TFMsin['3PP'] < LS_3P) & (TFMsin['3PP'] > LI_3P)]
TFMsin = TFMsin[(TFMsin['2P'] < LS_2P) & (TFMsin['2P'] > LI_2P)]
TFMsin = TFMsin[(TFMsin['2PA'] < LS_2P) & (TFMsin['2PA'] > LI_2P)]
TFMsin = TFMsin[(TFMsin['2PP'] < LS_2P) & (TFMsin['2PP'] > LI_2P)]
TFMsin = TFMsin[(TFMsin.eFGP < LS_eFGP) & (TFMsin.eFGP > LI_eFGP)]
TFMsin = TFMsin[(TFMsin.FT < LS_FT) & (TFMsin.FT > LI_FT)]
TFMsin = TFMsin[(TFMsin.FTA < LS_FTA) & (TFMsin.FTA > LI_FTA)]
TFMsin = TFMsin[(TFMsin.FTP < LS_FTP) & (TFMsin.FTP > LI_FTP)]
TFMsin = TFMsin[(TFMsin.PTS < LS_PTS) & (TFMsin.PTS > LI_PTS)]
TFMsin = TFMsin[(TFMsin.ORB < LS_ORB) & (TFMsin.ORB > LI_ORB)]
TFMsin = TFMsin[(TFMsin.DRB < LS_DRB) & (TFMsin.DRB > LI_DRB)]
TFMsin = TFMsin[(TFMsin.TRB < LS_TRB) & (TFMsin.TRB > LI_TRB)]
TFMsin = TFMsin[(TFMsin.AST < LS_AST) & (TFMsin.AST > LI_AST)]
TFMsin = TFMsin[(TFMsin.STL < LS_STL) & (TFMsin.STL > LI_STL)]
TFMsin = TFMsin[(TFMsin.BLK < LS_BLK) & (TFMsin.BLK > LI_BLK)]
TFMsin = TFMsin[(TFMsin.TOV < LS_TOV) & (TFMsin.TOV > LI_TOV)]
TFMsin = TFMsin[(TFMsin.PF < LS_PF) & (TFMsin.PF > LI_PF)]
TFMsin.shape
#7289 registros
#Reseteamos el indice para no tener problemas con los bucles
TFMsin=TFMsin.reset_index(drop=True)

###ANALISIS EXPLORATORIO
#Matriz correlacion de datos eliminando variables no numericas
Corr=TFM.iloc[:,16:39]
Corr=pd.concat([TFM.Altura,TFM.Peso,TFM.Edad,TFM.Minutos,TFM.PER,Corr], axis=1)

fig, ax = plt.subplots(figsize=(15,10))
sns.heatmap(Corr.corr(), annot=True, ax=ax)
Corr.corr()
#Se observan como las variables altura y peso estan fuertemente correladas
#La variable minutos con todas las variables de estadisticas del jugador
#O la variable PTS con las variables de anotacion
#Hay variables que son combinación de otras como las de porcentaje que quizás
#sería interesante aplicar una reduccion de dimensiones más adelante

#Diagrama de dispersion Peso-Altura
sns.scatterplot(data=TFM, x="Peso", y="Altura")
#Se observa que cuando mayor es el peso, mayor es la altura

#Diagrama de dispersion Peso-Altura por Posicion
sns.scatterplot(data=TFM, x="Peso", y="Altura",hue="Pos",
                hue_order=["Base","Escolta","Alero","Ala-Pivot","Pivot"]).set_title("Peso vs Altura por Posición")
#Los jugadores se ordenan por posicion quedando en los extremos los bases y pivots

#Histograma de altura
sns.histplot(data=TFM, x='Altura', bins=10, color='#F2AB6D')

#Histograma de altura por posicion
AlturaClas = []
for i in range(0,len(TFM)):
    if TFM.Altura[i]<180:
          AlturaClas.append('<180')
    elif TFM.Altura[i]>180 and TFM.Altura[i]<210:
          AlturaClas.append('>180 y <210' )     
    else:
        AlturaClas.append('>210')
        
AlturaData = list(zip(TFM.Altura,AlturaClas,TFM.Pos))
AlturaData = pd.DataFrame(AlturaData)
AlturaData.columns = ['Altura','Grupo','Pos']
AlturaDataOrder=AlturaData.sort_values('Grupo')
sns.histplot(data = AlturaDataOrder, x='Grupo', hue="Pos", 
             hue_order=["Base","Escolta","Alero","Ala-Pivot","Pivot"],
             multiple="dodge").set_title("Histograma de Altura por Posición")
#El grueso mayor de jugadores se encuentran entre 180 y 210 cm, mientras que
#por debajo de 180 solo existen bases y por encima de 210 destacan os pivots

#Histograma de peso
sns.histplot(data=TFM, x='Peso', bins=10, color='#F2AB6D')

#Histograma de peso por posicion
PesoClas = []
for i in range(0,len(TFM)):
    if TFM.Peso[i]<80:
         PesoClas.append('<80')
    elif TFM.Peso[i]>80 and TFM.Peso[i]<110:
          PesoClas.append('>080 y <110')     
    else:
        PesoClas.append('>110')
        
PesoData = list(zip( TFM.Peso,PesoClas,TFM.Pos))
PesoData = pd.DataFrame(PesoData)
PesoData.columns = ['Peso','Grupo','Pos']
PesoDataOrder=PesoData.sort_values('Grupo')
for i in range(0,len(PesoDataOrder)):
    if PesoDataOrder.Grupo[i]=='>080 y <110':
         PesoDataOrder.Grupo[i]='>80 y <110'  
    else:
        PesoDataOrder.Grupo[i]=PesoDataOrder.Grupo[i]
        
sns.histplot(data = PesoDataOrder, x='Grupo', hue="Pos", 
             hue_order=["Base","Escolta","Alero","Ala-Pivot","Pivot"],
             multiple="dodge").set_title("Histograma de Peso por Posición")
#El grueso mayor de jugadores se encuentran entre 180 y 210 cm, mientras que
#por debajo de 180 solo existen bases y por encima de 210 destacan os pivots

### Distribucion del resto de variables
TirosCampo=TFM.iloc[:,17:20]
sns.pairplot(TirosCampo)

TirosTres=TFM.iloc[:,20:23]
sns.pairplot(TirosTres)

TirosDos=TFM.iloc[:,23:26]
sns.pairplot(TirosDos)

TirosLibres=TFM.iloc[:,27:30]
sns.pairplot(TirosLibres)

Rebotes=TFM.iloc[:,31:34]
sns.pairplot(Rebotes)

RestoStats=TFM.iloc[:,34:39]
sns.pairplot(RestoStats)
#Se observa como existe informacion redundante
#Existen muchas variables que son combinacion de otras, por lo que van a ser eliminadas

###REDUCCION DE DIMENSIONES
#Eliminamos todas aquellas variables de lanzamientos, en las que dismonemos de
#tiros intentados y anotados, y nos quedamos con el porcentaje
TFM = TFM.drop(['FG','FGA','3P','3PA','2P','2PA','FT','FTA',
                'ORB','DRB'], axis=1)
TFMsin = TFMsin.drop(['FG','FGA','3P','3PA','2P','2PA','FT','FTA',
                'ORB','DRB'], axis=1)
#Finalmente disponemos de 29 variables
TFM.shape

#Boxplot Altura por posicion
sns.boxplot(x="Pos", y="Altura", data=TFM,
            order=["Base","Escolta","Alero","Ala-Pivot","Pivot"])

#Boxplot Peso por posicion
sns.boxplot(x="Pos", y="Peso", data=TFM,
            order=["Base","Escolta","Alero","Ala-Pivot","Pivot"])

#Numero de jugadores por posicion
sns.countplot(x="Pos", data=TFM,
              order=["Base","Escolta","Alero","Ala-Pivot","Pivot"])

#Numero de jugadores por equipo
sns.countplot(y="Equipo", data=TFM,
              order=TFM.Equipo.value_counts().iloc[:20].index)
#Los equipos con mas jugadores en nuestra bbdd son los sixers y los spurs

#Numero de jugadores por universidad
sns.countplot(y="Universidad", data=TFM,
              order=TFM.Universidad.value_counts().iloc[:10].index)
#Las dos universidades que mas jugadores han conseguido llevar a la nba son la
#de California y la de Carolina del Norte


###ANALISIS DESCRIPTIVO
#Se realiza un pequeño analisis descriptivo, se escogen los PTS, AST, RBT y BLK
Vis = TFM.iloc[:,[0,22,23,24,26]]
Vis = Vis.groupby(['Jugador']).sum()
Vis = Vis.rename_axis('Jugador').reset_index()

#Visualización: Puntos
dictionary={"Jugador":Vis.Jugador,"Puntos":Vis.PTS}
PuntosJugador=pd.DataFrame(dictionary)
PuntosJugador=PuntosJugador.sort_values("Puntos",ascending=False)

PuntosTop=PuntosJugador.head(20)

plt.figure(figsize=(20,13))
sns.barplot(x=PuntosTop.Jugador,y=PuntosTop.Puntos)
plt.xticks(rotation=60)
plt.ylabel("Puntos")
plt.xlabel("Jugador")
plt.title("NBA All Times Points Leaders")
plt.show()

#Visualización: Asistencias
dictionary={"Jugador":Vis.Jugador,"Asistencias":Vis.AST}
AsistenciasJugador=pd.DataFrame(dictionary)
AsistenciasJugador=AsistenciasJugador.sort_values("Asistencias",ascending=False)

AsistenciasTop=AsistenciasJugador.head(20)

plt.figure(figsize=(18,12))
sns.barplot(x=AsistenciasTop.Jugador,y=AsistenciasTop.Asistencias,palette =sns.cubehelix_palette(len(AsistenciasTop)))
plt.xticks(rotation=60)
plt.ylabel("Asistencias")
plt.xlabel("Jugador")
plt.title("NBA All Times Assists Leaders")
plt.show()

#Visualización: Rebotes
dictionary={"Jugador":Vis.Jugador,"Rebotes":Vis.TRB}
RebotesJugador=pd.DataFrame(dictionary)
RebotesJugador=RebotesJugador.sort_values("Rebotes",ascending=False)

RebotesTop=RebotesJugador.head(20)

plt.figure(figsize=(20,13))
sns.barplot(x=RebotesTop.Jugador,y=RebotesTop.Rebotes,palette=sns.cubehelix_palette(len(RebotesTop),rot=-.5))
plt.xticks(rotation=60)
plt.ylabel("Rebotes")
plt.xlabel("Jugador")
plt.title("NBA All Times Rebounds Leaders")
plt.show()

#Visualización: Tapones
dictionary={"Jugador":Vis.Jugador,"Tapones":Vis.BLK}
TaponesJugador=pd.DataFrame(dictionary)
TaponesJugador=TaponesJugador.sort_values("Tapones",ascending=False)

TaponesTop=TaponesJugador.head(20)

plt.figure(figsize=(18,12))
sns.barplot(x=TaponesTop.Jugador, y=TaponesTop.Tapones,palette =sns.color_palette("Reds_d",len(TaponesTop)))
plt.xticks(rotation=60)
plt.xlabel("Jugador")
plt.ylabel("Tapones")
plt.title("NBA All-Time Blocks Leaders")
plt.show()

#Visualización: Puntos por temporada
temporada=[]
totales=[]
for i in TFM.Año.unique():
    temporada.append(i)
    total=0
    x=TFM[TFM.Año==i]
    for j in x.PTS:
        total+=int(j)
    totales.append(total)  
Datos2=pd.DataFrame({"Temporada":temporada,"Puntos":totales})

plt.subplots(figsize =(20,10))
sns.pointplot(x="Temporada",y="Puntos",data=Datos2,color="red",alpha=1.2)
plt.xlabel("Temporada",fontsize = 25)
plt.ylabel("Puntos Totales",fontsize = 25)
plt.xticks(rotation=70)
plt.title("Puntos Totales por Temporada",fontsize = 30,color='blue')
plt.grid()
#Durante las temporadas 98-99 y 11-12, se produjó un cierre patronal que obligó
#a reducir la temporada regular a 50 y 62 partidos respectivamente

#Se divide el conjunto de datos por posicion
Bases=TFM[TFM.Pos=="Base"]
Escoltas=TFM[TFM.Pos=="Escolta"]
Aleros=TFM[TFM.Pos=="Alero"]
AlaPivots=TFM[TFM.Pos=="Ala-Pivot"]
Pivots=TFM[TFM.Pos=="Pivot"]

DatosBases=Bases.groupby("Año").sum()
DatosEscoltas=Escoltas.groupby("Año").sum()
DatosAleros=Aleros.groupby("Año").sum()
DatosAlaPivots=AlaPivots.groupby("Año").sum()
DatosPivots=Pivots.groupby("Año").sum()

#Visualizacion puntos por posicion
plt.subplots(figsize =(20,10))
plot1=sns.pointplot(x=DatosBases.index,y="PTS",data=DatosBases,color="blue",alpha=1.0,label="Base")
plot2=sns.pointplot(x=DatosEscoltas.index,y="PTS",data=DatosEscoltas,color="green",alpha=1.0,label="Escolta")
plot3=sns.pointplot(x=DatosAleros.index,y="PTS",data=DatosAleros,color="yellow",alpha=1.0,label="Alero")
plot4=sns.pointplot(x=DatosAlaPivots.index,y="PTS",data=DatosAlaPivots,color="orange",alpha=1.0,label="Ala-Pivot")
plot5=sns.pointplot(x=DatosPivots.index,y="PTS",data=DatosPivots,color="red",alpha=1.0,label="Pivot")
plt.xlabel("Temporada",fontsize = 25)
plt.ylabel("Puntos Totales",fontsize = 25)
plt.xticks(rotation=70)
plt.title("Puntos Totales por Temporada y Posición",fontsize = 30)
plt.text(5,30000,"Base",color="blue",fontsize =20)
plt.text(5,27500,"Escolta",color="green",fontsize=20)
plt.text(5,25000,"Alero",color="yellow",fontsize =20)
plt.text(5,22500,"Ala-Pivot",color="orange",fontsize=20)
plt.text(5,20000,"Pivot",color="red",fontsize =20)
plt.grid()

#Visualizacion asistencias por posicion
plt.subplots(figsize =(20,10))
plot1=sns.pointplot(x=DatosBases.index,y="AST",data=DatosBases,color="blue",alpha=1.0,label="Base")
plot2=sns.pointplot(x=DatosEscoltas.index,y="AST",data=DatosEscoltas,color="green",alpha=1.0,label="Escolta")
plot3=sns.pointplot(x=DatosAleros.index,y="AST",data=DatosAleros,color="yellow",alpha=1.0,label="Alero")
plot4=sns.pointplot(x=DatosAlaPivots.index,y="AST",data=DatosAlaPivots,color="orange",alpha=1.0,label="Ala-Pivot")
plot5=sns.pointplot(x=DatosPivots.index,y="AST",data=DatosPivots,color="red",alpha=1.0,label="Pivot")
plt.xlabel("Temporada",fontsize = 25)
plt.ylabel("Asistencias Totales",fontsize = 25)
plt.xticks(rotation=70)
plt.title("Asistencias Totales por Temporada y Posición",fontsize = 30)
plt.text(1,23000,"Base",color="blue",fontsize =20)
plt.text(1,22000,"Escolta",color="green",fontsize=20)
plt.text(1,21000,"Alero",color="yellow",fontsize =20)
plt.text(1,20000,"Ala-Pivot",color="orange",fontsize=20)
plt.text(1,19000,"Pivot",color="red",fontsize =20)
plt.grid()

#Visualizacion rebotes por posicion
plt.subplots(figsize =(20,10))
plot1=sns.pointplot(x=DatosBases.index,y="TRB",data=DatosBases,color="blue",alpha=1.0,label="Base")
plot2=sns.pointplot(x=DatosEscoltas.index,y="TRB",data=DatosEscoltas,color="green",alpha=1.0,label="Escolta")
plot3=sns.pointplot(x=DatosAleros.index,y="TRB",data=DatosAleros,color="yellow",alpha=1.0,label="Alero")
plot4=sns.pointplot(x=DatosAlaPivots.index,y="TRB",data=DatosAlaPivots,color="orange",alpha=1.0,label="Ala-Pivot")
plot5=sns.pointplot(x=DatosPivots.index,y="TRB",data=DatosPivots,color="red",alpha=1.0,label="Pivot")
plt.xlabel("Temporada",fontsize = 25)
plt.ylabel("Rebotes Totales",fontsize = 25)
plt.xticks(rotation=70)
plt.title("Rebotes Totales por Temporada y Posición",fontsize = 30)
plt.text(1,30000,"Base",color="blue",fontsize =20)
plt.text(1,29000,"Escolta",color="green",fontsize=20)
plt.text(1,28000,"Alero",color="yellow",fontsize =20)
plt.text(1,27000,"Ala-Pivot",color="orange",fontsize=20)
plt.text(1,26000,"Pivot",color="red",fontsize =20)
plt.grid()

#Visualizacion tapones por posicion
plt.subplots(figsize =(20,10))
plot1=sns.pointplot(x=DatosBases.index,y="BLK",data=DatosBases,color="blue",alpha=1.0,label="Base")
plot2=sns.pointplot(x=DatosEscoltas.index,y="BLK",data=DatosEscoltas,color="green",alpha=1.0,label="Escolta")
plot3=sns.pointplot(x=DatosAleros.index,y="BLK",data=DatosAleros,color="yellow",alpha=1.0,label="Alero")
plot4=sns.pointplot(x=DatosAlaPivots.index,y="BLK",data=DatosAlaPivots,color="orange",alpha=1.0,label="Ala-Pivot")
plot5=sns.pointplot(x=DatosPivots.index,y="BLK",data=DatosPivots,color="red",alpha=1.0,label="Pivot")
plt.xlabel("Temporada",fontsize = 25)
plt.ylabel("Tapones Totales",fontsize = 25)
plt.xticks(rotation=70)
plt.title("Tapones Totales por Temporada y Posición",fontsize = 30)
plt.text(1,5200,"Base",color="blue",fontsize =20)
plt.text(1,5000,"Escolta",color="green",fontsize=20)
plt.text(1,4800,"Alero",color="yellow",fontsize =20)
plt.text(1,4600,"Ala-Pivot",color="orange",fontsize=20)
plt.text(1,4400,"Pivot",color="red",fontsize =20)
plt.grid()

###IMPORTANCIA DE VARIABLES: RANDOM FOREST

#Creamos los dos conjunto de datos con y sin outliers
df=TFM.iloc[:,16:29]
df=pd.concat([TFM.Altura,TFM.Peso,TFM.Edad,TFM.Minutos,TFM.PER,df], axis=1)
df=pd.concat([df,TFM.Pos], axis=1)

dfsin=TFMsin.iloc[:,16:29]
dfsin=pd.concat([TFMsin.Altura,TFMsin.Peso,TFMsin.Edad,TFMsin.Minutos,TFMsin.PER,dfsin], axis=1)
dfsin=pd.concat([dfsin,TFMsin.Pos], axis=1)

#Convertimos la columna pos en numerica, ya que la utilizaremos para predecir
for i in range(0,len(df)):
    if df.Pos[i]=='Base':
          df.Pos[i]=0
    elif df.Pos[i]=='Escolta':
          df.Pos[i]=1
    elif df.Pos[i]=='Alero':
          df.Pos[i]=2
    elif df.Pos[i]=='Ala-Pivot':
          df.Pos[i]=3      
    else:
        df.Pos[i]=4
        
for i in range(0,len(dfsin)):
    if dfsin.Pos[i]=='Base':
          dfsin.Pos[i]=0
    elif dfsin.Pos[i]=='Escolta':
          dfsin.Pos[i]=1
    elif dfsin.Pos[i]=='Alero':
          dfsin.Pos[i]=2
    elif dfsin.Pos[i]=='Ala-Pivot':
          dfsin.Pos[i]=3      
    else:
        dfsin.Pos[i]=4

df = df.astype(str).astype(float)
dfsin = dfsin.astype(str).astype(float)

#Guardamos los encabezados de cada columna
headers = df.columns.values.tolist()
headers.remove('Pos')
        
#Creamos los conjuntos de train y test para el df con outliers
m_train     = np.random.rand(len(df)) < 0.8
data_train  = df.loc[m_train,headers].to_numpy()
data_test   = df.loc[~m_train,headers].to_numpy()
clase_train = df.loc[m_train,'Pos'].to_numpy()
clase_test  = df.loc[~m_train,'Pos'].to_numpy()

data_train = np.matrix(data_train)
data_test  = np.matrix(data_test) 

#Creamos los conjuntos de train y test para el df sin outliers
m_train2     = np.random.rand(len(dfsin)) < 0.8
data_train2  = dfsin.loc[m_train2,headers].to_numpy()
data_test2   = dfsin.loc[~m_train2,headers].to_numpy()
clase_train2 = dfsin.loc[m_train2,'Pos'].to_numpy()
clase_test2  = dfsin.loc[~m_train2,'Pos'].to_numpy()

data_train2 = np.matrix(data_train2)
data_test2  = np.matrix(data_test2) 

#Configuramos ambos modelos
from sklearn.ensemble import RandomForestClassifier 

modelo = RandomForestClassifier(
 random_state      = 1,   # semilla inicial de aleatoriedad del algoritmo
 n_estimators      = 666, # cantidad de arboles a crear
 min_samples_split = 2,   # cantidad minima de observaciones para dividir un nodo
 min_samples_leaf  = 1,   # observaciones minimas que puede tener una hoja del arbol
 n_jobs            = 1    # tareas en paralelo. para todos los cores disponibles usar -1
 )
modelo.fit(X = data_train, y = clase_train)

modelosin = RandomForestClassifier(
 random_state      = 1,   # semilla inicial de aleatoriedad del algoritmo
 n_estimators      = 666, # cantidad de arboles a crear
 min_samples_split = 2,   # cantidad minima de observaciones para dividir un nodo
 min_samples_leaf  = 1,   # observaciones minimas que puede tener una hoja del arbol
 n_jobs            = 1    # tareas en paralelo. para todos los cores disponibles usar -1
 )
modelosin.fit(X = data_train2, y = clase_train2)

# Observamos la importancia de cada variable en ambos modelos

#Con outliers
var_imp = pd.DataFrame({
 'feature':headers, 
 'v_importance':modelo.feature_importances_.tolist()
 })
var_imp.sort_values(by=['v_importance'],ascending=False)

#Sin outliers
var_imp = pd.DataFrame({
 'feature':headers, 
 'v_importance':modelosin.feature_importances_.tolist()
 })
var_imp.sort_values(by=['v_importance'],ascending=False)

#Solo existen dos cambios entre las 10 primeras, con outliers entran FGP y STL,
#mientras que sin outliers PF y FTP

#Reducimos ambos conjuntos a únicamente las 10 variables que mas peso tienen
df = df.drop(['eFGP','2PP','TSP','Edad','TOV',
                    'PER','FTP','PF'], axis=1)

dfsin = dfsin.drop(['eFGP','2PP','TSP','Edad','TOV',
                    'FGP','STL','PER'], axis=1)
#Ya disponemos de los conjuntos en perfecto estado para realizar la prediccion

#Antes vamos a realizar un pequeño analisis exploratorio para observar el nuevo df

#Matriz correlacion 
Corr=df.iloc[:,0:10]
fig, ax = plt.subplots(figsize=(15,10))
sns.heatmap(Corr.corr(), annot=True, ax=ax).set_title('Matriz de correlaciones')
Corr.corr()

#Distribucion de las variables 
fig, axs = plt.subplots(2, 2)
axs[0, 0].hist(dfsin.FTP)
axs[0, 0].set_title('Tiros libres')
axs[0, 1].hist(df.FGP)
axs[0, 1].set_title('Tiros de campo')
axs[1, 0].hist(df['3PP'])
axs[1, 0].set_title('Tiros de tres')
axs[1, 1].hist(df.PTS)
axs[1, 1].set_title('Puntos')
fig.tight_layout()

fig, axs = plt.subplots(2, 3)
axs[0, 0].hist(df.TRB)
axs[0, 0].set_title('Rebotes')
axs[0, 1].hist(df.AST)
axs[0, 1].set_title('Asistencias')
axs[0, 2].hist(df.STL)
axs[0, 2].set_title('Robos')
axs[1, 0].hist(df.BLK)
axs[1, 0].set_title('Tapones')
axs[1, 1].hist(dfsin.PF)
axs[1, 1].set_title('Faltas')
axs[1, 2].hist(df.BLK)
axs[1, 2].set_title('Minutos')
fig.tight_layout()


for ax in axs.flat:
    ax.set(xlabel='x-label', ylabel='y-label')

# Hide x labels and tick labels for top plots and y ticks for right plots.
for ax in axs.flat:
    ax.label_outer()

#PREDICCION DE POSICION POR ESTADISTICAS

###REGRESION LINEAL

#Librerias necesarias
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import MinMaxScaler
from sklearn import linear_model
import collections
from sklearn.metrics import confusion_matrix

#Creamos un conjunto X con todos los datos excepto la posicion, ya que queremos
#clasificar por ella para los datos con outliers
X = df.drop(['Pos'], axis=1)

#Creamos un conjunto y guardando la columna de posicion
y = df['Pos']

#Creamos los conjuntos de entrenamiento y test tanto de x como de y con la funcion
#train_test_split, fijamos la semilla en 42 y un tamaño del test del 20%
Xtrain, Xtest, ytrain, ytest = train_test_split(
    X, y, random_state=42, test_size=0.2)

Xtrain.shape
#El tamaño es de 10006 registros y 10 variables (80%)
Xtest.shape
#El tamaño es de 6672 registros y 10 variables (20%)

#Creamos los conjuntos para el df sin outliers
Xsin = dfsin.drop(['Pos'], axis=1)

ysin = dfsin['Pos']

#Creamos los conjuntos de entrenamiento y test tanto de x como de y con la funcion
#train_test_split, fijamos la semilla en 42 y un tamaño del test del 20%
Xtrain2, Xtest2, ytrain2, ytest2 = train_test_split(
    Xsin, ysin, random_state=42, test_size=0.2)

Xtrain2.shape
#El tamaño es de 4373 registros y 10 variables (80%)
Xtest2.shape
#El tamaño es de 2916 registros y 10 variables (20%)

#MODELO CON OUTLIERS

#Convertimos los valores a una escala de 0-1
scaler = MinMaxScaler()
X_train_reg = scaler.fit_transform(Xtrain)
X_test_reg = scaler.transform(Xtest)
y_train_reg = ytrain

model = linear_model.LinearRegression()
model.fit(X_train_reg, y_train_reg)

#Realizamos la prediccion sobre el test
y_pred = model.predict(X_test_reg)

# Redondeamos el resultado y convertimos en entero
y = np.rint(y_pred)
y = y.astype(int) 
res = np.hstack(y)

#Contamos las aparaciones de cada elemento
print(collections.Counter(res))

#Existen algunos valores negativos, por lo que los convertimos a cero, y
#los valores mayores que cuatro a 4
Reg_results = res.copy()
Reg_results[y_pred < 0] = 0
Reg_results[y_pred > 4] = 4
Reg_results

#Generamos el resultado
Resultado = pd.DataFrame({'Pos':ytest , 'Pos_Predict': Reg_results})

#Volvemos a renombrar los valores en ambas columnas
for i in range(0,len(Resultado)):
    if Resultado.iloc[i,0]==0:
          Resultado.iloc[i,0]='Base'
    elif Resultado.iloc[i,0]==1:
          Resultado.iloc[i,0]='Escolta'
    elif Resultado.iloc[i,0]==2:
          Resultado.iloc[i,0]='Alero'
    elif Resultado.iloc[i,0]==3:
          Resultado.iloc[i,0]='Ala-Pivot'     
    else:
        Resultado.iloc[i,0]='Pivot'

#Volvemos a renombrar los valores en ambas columnas
for i in range(0,len(Resultado)):
    if Resultado.iloc[i,1]==0:
          Resultado.iloc[i,1]='Base'
    elif Resultado.iloc[i,1]==1:
          Resultado.iloc[i,1]='Escolta'
    elif Resultado.iloc[i,1]==2:
          Resultado.iloc[i,1]='Alero'
    elif Resultado.iloc[i,1]==3:
          Resultado.iloc[i,1]='Ala-Pivot'     
    else:
        Resultado.iloc[i,1]='Pivot'
        
#Creamos la matriz de confusiom
cm = confusion_matrix(Resultado['Pos'],Resultado['Pos_Predict'])

#Observamos como van a existir cambios
Resultado['Pos'].value_counts()
Resultado['Pos_Predict'].value_counts()

#Plot
sns.heatmap(cm, annot=True,fmt="d",
            xticklabels=['Ala-Pivot','Alero','Base','Escolta','Pivot'], 
            yticklabels=['Ala-Pivot','Alero','Base','Escolta',
                         'Pivot']).set(title='Matriz de Confusion', xlabel='Valores reales', ylabel='Valores predichos')
#Existen problemas en casi todas las posiciones

contador=0
for i in range(0,len(Resultado)):
    if Resultado.iloc[i,1]==Resultado.iloc[i,0]:
          contador=contador+1   
    else:
        contador=contador 
proporcion=(contador/len(Resultado))*100
print('El modelo tiene un acierto del:'+" "+ str(round(proporcion,2)))

#MODELO SIN OUTLIERS

#Convertimos los valores a una escala de 0-1
scaler = MinMaxScaler()
X_train_reg = scaler.fit_transform(Xtrain2)
X_test_reg = scaler.transform(Xtest2)
y_train_reg = ytrain2

#Aplicamos el modelo
from sklearn import linear_model
model = linear_model.LinearRegression()
model.fit(X_train_reg, y_train_reg)

#Realizamos la prediccion sobre el test
y_pred = model.predict(X_test_reg)

# Redondeamos el resultado y convertimos en entero
import numpy as np
y = np.rint(y_pred)
y = y.astype(int) 
res = np.hstack(y)

#Contamos las aparaciones de cada elemento
import collections
print(collections.Counter(res))

#Existen algunos valores negativos, por lo que los convertimos a cero, y
#los valores mayores qe cuatro a cuatrp
Reg_results = res.copy()
Reg_results[y_pred < 0] = 0
Reg_results[y_pred > 4] = 4
Reg_results

#Generamos el resultado
Resultado2 = pd.DataFrame({'Pos':ytest2 , 'Pos_Predict': Reg_results})

#Volvemos a renombrar los valores en ambas columnas
for i in range(0,len(Resultado2)):
    if Resultado2.iloc[i,0]==0:
          Resultado2.iloc[i,0]='Base'
    elif Resultado2.iloc[i,0]==1:
          Resultado2.iloc[i,0]='Escolta'
    elif Resultado2.iloc[i,0]==2:
          Resultado2.iloc[i,0]='Alero'
    elif Resultado2.iloc[i,0]==3:
          Resultado2.iloc[i,0]='Ala-Pivot'     
    else:
        Resultado2.iloc[i,0]='Pivot'

#Volvemos a renombrar los valores en ambas columnas
for i in range(0,len(Resultado2)):
    if Resultado2.iloc[i,1]==0:
          Resultado2.iloc[i,1]='Base'
    elif Resultado2.iloc[i,1]==1:
          Resultado2.iloc[i,1]='Escolta'
    elif Resultado2.iloc[i,1]==2:
          Resultado2.iloc[i,1]='Alero'
    elif Resultado2.iloc[i,1]==3:
          Resultado2.iloc[i,1]='Ala-Pivot'     
    else:
        Resultado2.iloc[i,1]='Pivot'
        
#Creamos la matriz de confusiom
from sklearn.metrics import confusion_matrix
cm = confusion_matrix(Resultado2['Pos'],Resultado2['Pos_Predict'])

#Observamos como van a existir cambios
Resultado2['Pos'].value_counts()
Resultado2['Pos_Predict'].value_counts()

#Plot
sns.heatmap(cm, annot=True,fmt="d",
            xticklabels=['Ala-Pivot','Alero','Base','Escolta','Pivot'], 
            yticklabels=['Ala-Pivot','Alero','Base','Escolta',
                         'Pivot']).set(title='Matriz de Confusion', xlabel='Valores reales', ylabel='Valores predichos')
#Existen menos problemas, notables solo entre pivot y ala-pivot

contador=0
for i in range(0,len(Resultado2)):
    if Resultado2.iloc[i,1]==Resultado2.iloc[i,0]:
          contador=contador+1   
    else:
        contador=contador 
proporcion=(contador/len(Resultado2))*100
print('El modelo tiene un acierto del:'+" "+ str(round(proporcion,2)))

###KNN CON OUTLIERS
from sklearn import neighbors
from sklearn.model_selection import KFold
import matplotlib.pyplot as plt
from sklearn.metrics import mean_absolute_error
import numpy as np


trainKNN, testKNN = train_test_split(
    df, random_state=42, test_size=0.2)


trainKNN.reset_index(drop = True, inplace = True)
testKNN.reset_index(drop = True, inplace = True)

#Convertimos los valores a una escala de 0-1 excepto la ultima columna
scaler = MinMaxScaler()
trainKNNz = scaler.fit_transform(trainKNN.iloc[:,0:10])
testKNNz = scaler.transform(testKNN.iloc[:,0:10])
trainKNNz = pd.DataFrame(trainKNNz)
trainKNN=pd.concat([trainKNNz,trainKNN.Pos], axis=1)
testKNNz = pd.DataFrame(testKNNz)
testKNN=pd.concat([testKNNz,testKNN.Pos], axis=1)


cv = KFold(n_splits = 10, shuffle = False) 

for i, weights in enumerate(['uniform', 'distance']):
   total_scores = []
   for n_neighbors in range(1,30):
       fold_accuracy = []
       knn = neighbors.KNeighborsRegressor(n_neighbors, weights=weights)
       # verificar cada uno de los modelos con validación cruzada.
       for train_fold, test_fold in cv.split(trainKNN):
          # División train test aleatoria
          f_train = trainKNN.loc[train_fold]
          f_test = trainKNN.loc[test_fold]
          # entrenamiento y ejecución del modelo
          knn.fit( X = f_train.drop(['Pos'], axis=1), 
                               y = f_train['Pos'])
          y_pred = knn.predict(X = f_test.drop(['Pos'], axis = 1))
          # evaluación del modelo
          mae = mean_absolute_error(f_test['Pos'], y_pred)
          fold_accuracy.append(mae)
       total_scores.append(sum(fold_accuracy)/len(fold_accuracy))
   
   plt.plot(range(1,len(total_scores)+1), total_scores, 
             marker='o', label=weights)
   print ('Min Value ' +  weights + " : " +  str(min(total_scores)) +" (" + str(np.argmin(total_scores) + 1) + ")")
   plt.ylabel('MAE')      
    
plt.legend()
plt.show() 


n_neighbors = 20
weights = 'distance'
knn = neighbors.KNeighborsRegressor(n_neighbors= n_neighbors, weights=weights) 


knn.fit( X = trainKNN.drop(['Pos'], axis=1), y = trainKNN['Pos'])
y_pred = knn.predict(X = testKNN.drop(['Pos'], axis = 1))
mae = mean_absolute_error(testKNN['Pos'], y_pred)
print ('MAE', mae)

import numpy as np
y = np.rint(y_pred)
y = y.astype(int) 
res = np.hstack(y)

#Contamos las aparaciones de cada elemento
import collections
print(collections.Counter(res))

KNN_results = res.copy()

#Generamos el resultado
Resultado3 = pd.DataFrame({'Pos':testKNN.Pos , 'Pos_Predict': KNN_results})

#Volvemos a renombrar los valores en ambas columnas
for i in range(0,len(Resultado3)):
    if Resultado3.iloc[i,0]==0:
          Resultado3.iloc[i,0]='Base'
    elif Resultado3.iloc[i,0]==1:
          Resultado3.iloc[i,0]='Escolta'
    elif Resultado3.iloc[i,0]==2:
          Resultado3.iloc[i,0]='Alero'
    elif Resultado3.iloc[i,0]==3:
          Resultado3.iloc[i,0]='Ala-Pivot'     
    else:
        Resultado3.iloc[i,0]='Pivot'

#Volvemos a renombrar los valores en ambas columnas
for i in range(0,len(Resultado3)):
    if Resultado3.iloc[i,1]==0:
          Resultado3.iloc[i,1]='Base'
    elif Resultado3.iloc[i,1]==1:
          Resultado3.iloc[i,1]='Escolta'
    elif Resultado3.iloc[i,1]==2:
          Resultado3.iloc[i,1]='Alero'
    elif Resultado3.iloc[i,1]==3:
          Resultado3.iloc[i,1]='Ala-Pivot'     
    else:
        Resultado3.iloc[i,1]='Pivot'
        
#Creamos la matriz de confusiom
from sklearn.metrics import confusion_matrix
cm = confusion_matrix(Resultado3['Pos'],Resultado3['Pos_Predict'])

#Observamos como van a existir cambios
Resultado3['Pos'].value_counts()
Resultado3['Pos_Predict'].value_counts()

#Plot
sns.heatmap(cm, annot=True,fmt="d",
            xticklabels=['Ala-Pivot','Alero','Base','Escolta','Pivot'], 
            yticklabels=['Ala-Pivot','Alero','Base','Escolta',
                         'Pivot']).set(title='Matriz de Confusion', xlabel='Valores reales', ylabel='Valores predichos')
#Comprobamos el porcentaje de acierto
contador=0
for i in range(0,len(Resultado3)):
    if Resultado3.iloc[i,1]==Resultado3.iloc[i,0]:
          contador=contador+1   
    else:
        contador=contador 
proporcion=(contador/len(Resultado3))*100
print('El modelo tiene un acierto del:'+" "+ str(round(proporcion,2)))

###KNN SIN OUTLIERS
trainKNN2, testKNN2 = train_test_split(
    dfsin, random_state=42, test_size=0.2)

trainKNN2.reset_index(drop = True, inplace = True)
testKNN2.reset_index(drop = True, inplace = True)

#Convertimos los valores a una escala de 0-1
scaler = MinMaxScaler()
trainKNN2z = scaler.fit_transform(trainKNN2.iloc[:,0:10])
testKNN2z = scaler.transform(testKNN2.iloc[:,0:10])
trainKNN2z = pd.DataFrame(trainKNN2z)
trainKNN2=pd.concat([trainKNN2z,trainKNN2.Pos], axis=1)
testKNN2z = pd.DataFrame(testKNN2z)
testKNN2=pd.concat([testKNN2z,testKNN2.Pos], axis=1)

cv = KFold(n_splits = 10, shuffle = False) 

for i, weights in enumerate(['uniform', 'distance']):
   total_scores = []
   for n_neighbors in range(1,30):
       fold_accuracy = []
       knn = neighbors.KNeighborsRegressor(n_neighbors, weights=weights)
       # verificar cada uno de los modelos con validación cruzada.
       for train_fold, test_fold in cv.split(trainKNN2):
          # División train test aleatoria
          f_train = trainKNN2.loc[train_fold]
          f_test = trainKNN2.loc[test_fold]
          # entrenamiento y ejecución del modelo
          knn.fit( X = f_train.drop(['Pos'], axis=1), 
                               y = f_train['Pos'])
          y_pred = knn.predict(X = f_test.drop(['Pos'], axis = 1))
          # evaluación del modelo
          mae = mean_absolute_error(f_test['Pos'], y_pred)
          fold_accuracy.append(mae)
       total_scores.append(sum(fold_accuracy)/len(fold_accuracy))
   
   plt.plot(range(1,len(total_scores)+1), total_scores, 
             marker='o', label=weights)
   print ('Min Value ' +  weights + " : " +  str(min(total_scores)) +" (" + str(np.argmin(total_scores) + 1) + ")")
   plt.ylabel('MAE')      
    
plt.legend()
plt.show() 


n_neighbors = 20
weights = 'distance'
knn = neighbors.KNeighborsRegressor(n_neighbors= n_neighbors, weights=weights) 


knn.fit( X = trainKNN2.drop(['Pos'], axis=1), y = trainKNN2['Pos'])
y_pred = knn.predict(X = testKNN2.drop(['Pos'], axis = 1))
mae = mean_absolute_error(testKNN2['Pos'], y_pred)
print ('MAE', mae)

import numpy as np
y = np.rint(y_pred)
y = y.astype(int) 
res = np.hstack(y)

#Contamos las aparaciones de cada elemento
import collections
print(collections.Counter(res))

KNN_results2 = res.copy()

#Generamos el resultado
Resultado4 = pd.DataFrame({'Pos':testKNN2.Pos , 'Pos_Predict': KNN_results2})

#Volvemos a renombrar los valores en ambas columnas
for i in range(0,len(Resultado4)):
    if Resultado4.iloc[i,0]==0:
          Resultado4.iloc[i,0]='Base'
    elif Resultado4.iloc[i,0]==1:
          Resultado4.iloc[i,0]='Escolta'
    elif Resultado4.iloc[i,0]==2:
          Resultado4.iloc[i,0]='Alero'
    elif Resultado4.iloc[i,0]==3:
          Resultado4.iloc[i,0]='Ala-Pivot'     
    else:
        Resultado4.iloc[i,0]='Pivot'

#Volvemos a renombrar los valores en ambas columnas
for i in range(0,len(Resultado4)):
    if Resultado4.iloc[i,1]==0:
          Resultado4.iloc[i,1]='Base'
    elif Resultado4.iloc[i,1]==1:
          Resultado4.iloc[i,1]='Escolta'
    elif Resultado4.iloc[i,1]==2:
          Resultado4.iloc[i,1]='Alero'
    elif Resultado4.iloc[i,1]==3:
          Resultado4.iloc[i,1]='Ala-Pivot'     
    else:
        Resultado4.iloc[i,1]='Pivot'
        
#Creamos la matriz de confusiom
from sklearn.metrics import confusion_matrix
cm = confusion_matrix(Resultado4['Pos'],Resultado4['Pos_Predict'])

#Observamos como van a existir cambios
Resultado4['Pos'].value_counts()
Resultado4['Pos_Predict'].value_counts()

#Plot
sns.heatmap(cm, annot=True,fmt="d",
            xticklabels=['Ala-Pivot','Alero','Base','Escolta','Pivot'], 
            yticklabels=['Ala-Pivot','Alero','Base','Escolta',
                         'Pivot']).set(title='Matriz de Confusion', xlabel='Valores reales', ylabel='Valores predichos')
#Comprobamos el porcentaje de acierto
contador=0
for i in range(0,len(Resultado4)):
    if Resultado4.iloc[i,1]==Resultado4.iloc[i,0]:
          contador=contador+1   
    else:
        contador=contador 
proporcion=(contador/len(Resultado4))*100
print('El modelo tiene un acierto del:'+" "+ str(round(proporcion,2)))

###RANDOM FOREST CON OUTLIERS

#Realizamos la prediccion del modelo realizado anteriormente
prediccion = modelo.predict(data_test)
prediccion = prediccion.astype(int) 
res = np.hstack(prediccion)

#Contamos las aparaciones de cada elemento
print(collections.Counter(res))
RF_results = res.copy()

#Generamos el resultado
Resultado5 = pd.DataFrame({'Pos':clase_test , 'Pos_Predict': RF_results})

#Volvemos a renombrar los valores en ambas columnas
for i in range(0,len(Resultado5)):
    if Resultado5.iloc[i,0]==0:
          Resultado5.iloc[i,0]='Base'
    elif Resultado5.iloc[i,0]==1:
          Resultado5.iloc[i,0]='Escolta'
    elif Resultado5.iloc[i,0]==2:
          Resultado5.iloc[i,0]='Alero'
    elif Resultado5.iloc[i,0]==3:
          Resultado5.iloc[i,0]='Ala-Pivot'     
    else:
        Resultado5.iloc[i,0]='Pivot'

#Volvemos a renombrar los valores en ambas columnas
for i in range(0,len(Resultado5)):
    if Resultado5.iloc[i,1]==0:
          Resultado5.iloc[i,1]='Base'
    elif Resultado5.iloc[i,1]==1:
          Resultado5.iloc[i,1]='Escolta'
    elif Resultado5.iloc[i,1]==2:
          Resultado5.iloc[i,1]='Alero'
    elif Resultado5.iloc[i,1]==3:
          Resultado5.iloc[i,1]='Ala-Pivot'     
    else:
        Resultado5.iloc[i,1]='Pivot'
        
#Creamos la matriz de confusiom
from sklearn.metrics import confusion_matrix
cm = confusion_matrix(Resultado5['Pos'],Resultado5['Pos_Predict'])

#Observamos como van a existir cambios
Resultado5['Pos'].value_counts()
Resultado5['Pos_Predict'].value_counts()

#Plot
sns.heatmap(cm, annot=True,fmt="d",
            xticklabels=['Ala-Pivot','Alero','Base','Escolta','Pivot'], 
            yticklabels=['Ala-Pivot','Alero','Base','Escolta',
                         'Pivot']).set(title='Matriz de Confusion', xlabel='Valores reales', ylabel='Valores predichos')
#Comprobamos el porcentaje de acierto
contador=0
for i in range(0,len(Resultado5)):
    if Resultado5.iloc[i,1]==Resultado5.iloc[i,0]:
          contador=contador+1   
    else:
        contador=contador 
proporcion=(contador/len(Resultado5))*100
print('El modelo tiene un acierto del:'+" "+ str(round(proporcion,2)))

###RANDOM FOREST SIN OUTLIERS

prediccion = modelosin.predict(data_test2)
prediccion = prediccion.astype(int) 
res = np.hstack(prediccion)

#Contamos las aparaciones de cada elemento
import collections
print(collections.Counter(res))
RF_results2 = res.copy()

#Generamos el resultado
Resultado6 = pd.DataFrame({'Pos':clase_test2 , 'Pos_Predict': RF_results2})

#Volvemos a renombrar los valores en ambas columnas
for i in range(0,len(Resultado6)):
    if Resultado6.iloc[i,0]==0:
          Resultado6.iloc[i,0]='Base'
    elif Resultado6.iloc[i,0]==1:
          Resultado6.iloc[i,0]='Escolta'
    elif Resultado6.iloc[i,0]==2:
          Resultado6.iloc[i,0]='Alero'
    elif Resultado6.iloc[i,0]==3:
          Resultado6.iloc[i,0]='Ala-Pivot'     
    else:
        Resultado6.iloc[i,0]='Pivot'

#Volvemos a renombrar los valores en ambas columnas
for i in range(0,len(Resultado6)):
    if Resultado6.iloc[i,1]==0:
          Resultado6.iloc[i,1]='Base'
    elif Resultado6.iloc[i,1]==1:
          Resultado6.iloc[i,1]='Escolta'
    elif Resultado6.iloc[i,1]==2:
          Resultado6.iloc[i,1]='Alero'
    elif Resultado6.iloc[i,1]==3:
          Resultado6.iloc[i,1]='Ala-Pivot'     
    else:
        Resultado6.iloc[i,1]='Pivot'
        
#Creamos la matriz de confusiom
from sklearn.metrics import confusion_matrix
cm = confusion_matrix(Resultado6['Pos'],Resultado6['Pos_Predict'])

#Observamos como van a existir cambios
Resultado6['Pos'].value_counts()
Resultado6['Pos_Predict'].value_counts()

#Plot
sns.heatmap(cm, annot=True,fmt="d",
            xticklabels=['Ala-Pivot','Alero','Base','Escolta','Pivot'], 
            yticklabels=['Ala-Pivot','Alero','Base','Escolta',
                         'Pivot']).set(title='Matriz de Confusion', xlabel='Valores reales', ylabel='Valores predichos')
#Comprobamos el porcentaje de acierto
contador=0
for i in range(0,len(Resultado6)):
    if Resultado6.iloc[i,1]==Resultado6.iloc[i,0]:
          contador=contador+1   
    else:
        contador=contador 
proporcion=(contador/len(Resultado6))*100
print('El modelo tiene un acierto del:'+" "+ str(round(proporcion,2)))

### Modelo XGBoost CON OUTLIERS

#Librerías necesarias
import xgboost as xgb
from sklearn.metrics import accuracy_score

#Creamos un conjunto y guardando la columna de posicion
y = df['Pos']

#Creamos los conjuntos de datos
XtrainXGB, XtestXGB, ytrainXGB, ytestXGB = train_test_split(
    X, y, random_state=42, test_size=0.2)
# Construyo el modelo y ajusto los datos.
xgboost = xgb.XGBClassifier()
xgboost.fit(XtrainXGB, ytrainXGB)

# Realizo las predicciones
y_pred = xgboost.predict(XtrainXGB)
y_pred = y_pred.astype(int)
predicciones = [round(value) for value in y_pred]
# Evalúo las predicciones
precision_train = accuracy_score(ytrainXGB, predicciones)

# Repito el proceso con datos de evaluacion
y_pred = xgboost.predict(XtestXGB)
y_pred = y_pred.astype(int)
predicciones = [round(value) for value in y_pred]

# Evalúo las predicciones
precision_test = accuracy_score(ytestXGB, predicciones)
print(xgboost)
print('Precisión xgboost train/test  {0:.3f}/{1:.3f}'
      .format(precision_train, precision_test))

#Generamos el resultado
Resultado7 = pd.DataFrame({'Pos':ytestXGB , 'Pos_Predict': y_pred})

#Volvemos a renombrar los valores en ambas columnas
for i in range(0,len(Resultado7)):
    if Resultado7.iloc[i,0]==0:
          Resultado7.iloc[i,0]='Base'
    elif Resultado7.iloc[i,0]==1:
          Resultado7.iloc[i,0]='Escolta'
    elif Resultado7.iloc[i,0]==2:
          Resultado7.iloc[i,0]='Alero'
    elif Resultado7.iloc[i,0]==3:
          Resultado7.iloc[i,0]='Ala-Pivot'     
    else:
        Resultado7.iloc[i,0]='Pivot'

#Volvemos a renombrar los valores en ambas columnas
for i in range(0,len(Resultado7)):
    if Resultado7.iloc[i,1]==0:
          Resultado7.iloc[i,1]='Base'
    elif Resultado7.iloc[i,1]==1:
          Resultado7.iloc[i,1]='Escolta'
    elif Resultado7.iloc[i,1]==2:
          Resultado7.iloc[i,1]='Alero'
    elif Resultado7.iloc[i,1]==3:
          Resultado7.iloc[i,1]='Ala-Pivot'     
    else:
        Resultado7.iloc[i,1]='Pivot'
        
#Creamos la matriz de confusiom
from sklearn.metrics import confusion_matrix
cm = confusion_matrix(Resultado7['Pos'],Resultado7['Pos_Predict'])

#Observamos como van a existir cambios
Resultado7['Pos'].value_counts()
Resultado7['Pos_Predict'].value_counts()

#Plot
sns.heatmap(cm, annot=True,fmt="d",
            xticklabels=['Ala-Pivot','Alero','Base','Escolta','Pivot'], 
            yticklabels=['Ala-Pivot','Alero','Base','Escolta',
                         'Pivot']).set(title='Matriz de Confusion', xlabel='Valores reales', ylabel='Valores predichos')
#Comprobamos el porcentaje de acierto
contador=0
for i in range(0,len(Resultado7)):
    if Resultado7.iloc[i,1]==Resultado7.iloc[i,0]:
          contador=contador+1   
    else:
        contador=contador 
proporcion=(contador/len(Resultado7))*100
print('El modelo tiene un acierto del:'+" "+ str(round(proporcion,2)))

### Modelo XGBoost SIN OUTLIERS

#Creamos los conjuntos de datos
XtrainXGB2, XtestXGB2, ytrainXGB2, ytestXGB2 = train_test_split(
    Xsin, ysin, random_state=42, test_size=0.2)
# Construyo el modelo y ajusto los datos.
xgboost2 = xgb.XGBClassifier()
xgboost2.fit(XtrainXGB2, ytrainXGB2)

# Realizo las predicciones
y_pred = xgboost2.predict(XtrainXGB2)
y_pred = y_pred.astype(int)
predicciones = [round(value) for value in y_pred]
# Evalúo las predicciones
precision_train = accuracy_score(ytrainXGB2, predicciones)

# Repito el proceso con datos de evaluacion
y_pred = xgboost2.predict(XtestXGB2)
y_pred = y_pred.astype(int)
predicciones = [round(value) for value in y_pred]

# Evalúo las predicciones
precision_test = accuracy_score(ytestXGB2, predicciones)
print(xgboost2)
print('Precisión xgboost train/test  {0:.3f}/{1:.3f}'
      .format(precision_train, precision_test))

#Generamos el resultado
Resultado8 = pd.DataFrame({'Pos':ytestXGB2 , 'Pos_Predict': y_pred})

#Volvemos a renombrar los valores en ambas columnas
for i in range(0,len(Resultado8)):
    if Resultado8.iloc[i,0]==0:
          Resultado8.iloc[i,0]='Base'
    elif Resultado8.iloc[i,0]==1:
          Resultado8.iloc[i,0]='Escolta'
    elif Resultado8.iloc[i,0]==2:
          Resultado8.iloc[i,0]='Alero'
    elif Resultado8.iloc[i,0]==3:
          Resultado8.iloc[i,0]='Ala-Pivot'     
    else:
        Resultado8.iloc[i,0]='Pivot'

#Volvemos a renombrar los valores en ambas columnas
for i in range(0,len(Resultado8)):
    if Resultado8.iloc[i,1]==0:
          Resultado8.iloc[i,1]='Base'
    elif Resultado8.iloc[i,1]==1:
          Resultado8.iloc[i,1]='Escolta'
    elif Resultado8.iloc[i,1]==2:
          Resultado8.iloc[i,1]='Alero'
    elif Resultado8.iloc[i,1]==3:
          Resultado8.iloc[i,1]='Ala-Pivot'     
    else:
        Resultado8.iloc[i,1]='Pivot'
        
#Creamos la matriz de confusiom
from sklearn.metrics import confusion_matrix
cm = confusion_matrix(Resultado8['Pos'],Resultado8['Pos_Predict'])

#Observamos como van a existir cambios
Resultado8['Pos'].value_counts()
Resultado8['Pos_Predict'].value_counts()

#Plot
sns.heatmap(cm, annot=True,fmt="d",
            xticklabels=['Ala-Pivot','Alero','Base','Escolta','Pivot'], 
            yticklabels=['Ala-Pivot','Alero','Base','Escolta',
                         'Pivot']).set(title='Matriz de Confusion', xlabel='Valores reales', ylabel='Valores predichos')
#Comprobamos el porcentaje de acierto
contador=0
for i in range(0,len(Resultado8)):
    if Resultado8.iloc[i,1]==Resultado8.iloc[i,0]:
          contador=contador+1   
    else:
        contador=contador 
proporcion=(contador/len(Resultado8))*100
print('El modelo tiene un acierto del:'+" "+ str(round(proporcion,2)))
