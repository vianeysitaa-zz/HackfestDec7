# Hackfest SQL Azure #
En esta sesión, aprenderemos cómo migrar una base de datos local a SQL Azure, hacer un modelo de predicción con Azure ML y visualizarlos en PowerBI

## Pre-requisitos ##
* Subscripción activa de Azure
* Contar con SQL Server instalado. Si no cuentas con SQL Server, puedes crear una máquina virtual con SQL instalado.
* Contar con Azure PowerShell instalado. Lo puedes descargar de la siguiente liga: [Instalación de Azure Powershell](https://docs.microsoft.com/es-mx/powershell/azure/install-azurerm-ps?view=azurermps-5.0.0)
* Contar con R y RStudio instalado. Pueden descargar la versión gratis de aquí: [Descargar R Studio](https://www.rstudio.com/products/rstudio/download/) 
* Instalar [Microsoft ODBC Driver](https://www.microsoft.com/en-us/download/details.aspx?id=36434)
* Instalar [Microsoft command Line Utilities](https://www.microsoft.com/en-us/download/details.aspx?id=36433)

## Fase 1 - Migración de Base de datos a la nube ##
En esta fase, vamos a crear una base de datos local con datos de prueba, y vamos a hacer una migración a la nube.

1. Abrir SQL Server Management Studio y concectarse a sus ervidor local.
2. Crear una base de datos con el nombre "TitanicDB".
3. Descargar el siguiente archivo csv que tiene la información prueba con la que vamos a trabajar. [Titanic info](https://microsoft-my.sharepoint.com/:f:/p/vianej/Et_TgCHYa6VKvRg8LSE1tFgBj0WalO380zPZDzvBnsM7Dw?e=02369954f08743df9c93ac780ced2b93)
4. Regresar a SQL Server Management Studio y dar clic derecho en nuestra base de datos "TitanicDB".
5. Dar clic en "Tasks" y dar clic en "Import Flat File".
6. Seleccionar la ruta del archivo que acabamos de descargar y escoger el noombre de la tabla TitanicData
![TitanicData](/images/Step6.png)
7. Dar clic en "Next" en los siguientes pasos (Preview Data, Modify Columns, Summary, Result).
8. Una vez que la información se haya importado, aparecerá un resultado de "Success".
> Ya tenemos  una base de datos con una tabla e información de prueba, ahora vamos a migrarla a la nube.
Existen diferentes procesos para hacer la migración. Escoger un método depende de los requerimientos. Por ejemplo, si no tenemos problema con un tiempo de inactividad de nuestra base de datos, podemos hacer la migración mediante el asistente "Data Migration Assistant".
Si un downtime no es una posibilidad, podemos realizar la migración mediante el método de replicación transaccional. Pueden encontrar más información sobre estos métodos en la siguiente liga:
[Migración de una base de datos de SQL Server a una instancia en SQL Azure](https://docs.microsoft.com/es-mx/azure/sql-database/sql-database-cloud-migrate)

> Para esta actividad, nosotros vamos a utilizar el asistente de migración.
Lo primero que tenemos que hacer, es crear nuestro servidor y base de datos en Azure.

9. Ingresar al portal de Azure y buscar "SQL Server (logical server)" y configurarlo.
![TitanicData](/images/Step8.png)

>Para poder acceder a las bases de datos que incluyamos en este servidor lógico, necesitamos configurar nuestro firewall. 
10. Dar clic en la opción "Firewall/Virtual Networks" y agregar nuestra dirección IP para poder conectarnos a el.
![TitanicData](/images/Step9.png)

>Ahora tenemos que crear la base de datos a la que migraremos el esquema y los datos. Esta base se creará en el servidor lógico que acabamos de desplegar.
11. En el portal de Azure, buscar "SQL Database" y configurarlo.
![TitanicData](/images/Step10.png)
12. Una vez que se haya generado nuestra base de datos, obtendremos el nombre del servidor para conectarnos posteriormente con la herramienta de migración.
![TitanicData](/images/Step10.png)

13. Descargar e instalar el asistente de migración de la siguiente liga: [Asistente de migración](https://www.microsoft.com/en-us/download/confirmation.aspx?id=53595)
14. Cuando se haya instalado la herramienta, vamos a agregar un nuevo proyecto de Migración.
![TitanicData](/images/Step11.png)
15. Conectárnos a nuestro servidor local. Elegir las opciones de "Encriptar información" y "Confiar en el certificado del servidor"
![TitanicData](/images/Step12.png)
16. Seleccionar la base de datos que queremos migrar.
![TitanicData](/images/Step13.png)
17. Seleccionar los objetos a migrar (En este caso solo tenemos 1 tabla)
![TitanicData](/images/Step14.png)
18. Conectarnos a nuestro servidor en Azure. Dar clic en "Deploy Schema" y por último en "Migrate data". (El nombre del servidor lo obtuvimos en el paso 12)
![TitanicData](/images/Step15.png)
> Una vez que se hayan migrado los datos, nos aparecerá un mensaje de éxito
![TitanicData](/images/Step16.png)
19. Ahora nos podemos conectar a la base de datos en Azure a través de SQL Server Management Studio y verificar que la tabla y los datos se encuentren ahí.

> Ahora que tenemos nuestra base de datos en la nube, vamos a generar un modelo de predicción sobre la probabilidad de supervivencia de un pasajero dependiendo de diferentes variables en nuestro set de datos.

## Fase 2 - Generación de un modelo de predicción ##

20. En nuestra base de datos en Azure, vamos a agregar una tabla con el mismo esquema que nuestra tabla TitanicData, pero agregando el campo "ScoredProbabilities".
En esta tabla vamos a almacenar la información proveniente de Azure ML con las probabilidades de supervivencia de los pasajeros. El esquema de la tabla es el siguiente:
```mysql
CREATE TABLE [dbo].[TitanicDataScored](
	[PassengerId] [int] NOT NULL,
	[Survived] [int] NULL,
	[Pclass] [int] NULL,
	[Name] [varchar](150) NULL,
	[Sex] [varchar](150) NULL,
	[Age] [int] NULL,
	[SibSp] [int] NULL,
	[Parch] [int] NULL,
	[Ticket] [varchar](150) NULL,
	[Fare] [decimal](10, 6) NULL,
	[Cabin] [varchar](10) NULL,
	[Embarked] [varchar](150) NULL,
	[ScoredProbabilities] [decimal](10, 6) NULL
) ON [PRIMARY]
GO
```

21. Ya que tenemos esta tabla, vamos a Azure ML Studio (https://studio.azureml.net/) e ingresamos con nuestras credenciales de Azure.
22. En Azure ML Studio, vamos a seleccionar un Expermiento en Blanco
![TitanicData](/images/Step22.png)
23. Renombramos el modelo a "Titanic Prediction".
> Ahora, vamos a agregar nuestro set de información para poder entrenar a nuestro modelo. (Tomaremos como fuente de datos el CSV TitanicData.csv)
24. En la esquina inferior izquierda dar clic en "+ New". Seleccionar la opción "DATASET" y por último "From Local File".
![TitanicData](/images/Step24.png)
25. En el formulario que aparece, seleccionar nuestro archivo local de TitanicData.csv, renombrar el DATASET a "TitanicData" y aceptar.
26. Una vez que nuestra información se haya cargado, en el menú izquierdo seleccionamos "Saved Data Sets", "My Data Sets" y arrastramos TitanicData al lienzo.
![TitanicData](/images/Step26.png)

> Los siguientes pasos serán para limpiar nuestra información

27. En el menú izquierdo, dar clic en Data Transformation, Manipulation y buscar "Select Columns in Dataset". Arrastrarlo al lienzo bajo nuestra fuente de datos.
28. Conectar los módulos. En el menú derecho dar clic en "Launch Column Selector".
![TitanicData](/images/Step27.png)
29. Configurar el módulo para excluir las columnas "PassengerId, Name, Cabin, Ticket". (Estas columnas se excluyen ya que no se consideran como una variable que pueda aportar al modelo una decisión o probabilidad de si el pasajero sobreviviría o no).
![TitanicData](/images/Step29.png)

30. Dentro del menu izquierdo Manipulation en Data Transformation, buscar el módulo "Edit Metadata", agregarlo al lienzo y conectarlo con el módulo anterior.
31. Dar clic en "Launch Column Selector" y seleccionar las columnas SibSp, Parch y Fare. Estas son las columnas que vamos a renombrar.
![TitanicData](/images/Step31.png)
32. En el menú derecho, vamos a colocar los nuevos nombres de las columnas: SiblingSpouse, ParentChild, FarePrice
![TitanicData](/images/Step32.png)
33. Agregamos y conectamos otro módulo de "Edit Metadata" y esta vez seleccionamos las columnas: Survived, Pclass, Sex, Embarked.
![TitanicData](/images/Step33.png)
34. Esta vez no solo vamos a renombar las columnas por Survived, PassengerClass, Gender, PortEmbarkation, sino que las vamos a convertir en variables categóricas.
![TitanicData](/images/Step34.png)
35. Agregamos y conectamos el módulo "Clean Missing Data". Escogemos todas las columnas y el modo de limpieza lo escogemos como "Remove entire row". Con esto eliminaremos las columnas que no tienen información y pueden perjudicar al entrenamiento del modelo.
![TitanicData](/images/Step35.png)
36. Dentro de Data Transformation, Sample and Split. Agregar y conectar el módulo "Split Data".
37. En las propiedades del módulo, ingresamos el valor de 0.8 a "Fraction of Rows". Esto nos servirá para dividir nuestra información en un 80% para entrenar el modelo y en 20% para poder evaluarlo.
![TitanicData](/images/Step37.png)
38. En el menú izquierdo, dentro de Machine Learning, Initialize Model, Classification, agregar al lienzo el módulo "Two-class boosted decission tree"
![TitanicData](/images/Step38.png)
> Escogimos este módulo porque vamos a clasificar si un pasajero sobrevivió o no. Para conocer los demás modelos pre-cargados en Azure ML, puedes descargar de esta liga un PDF con los modelos y cuándo los puedes usar. [Azure ML Cheat Sheet](https://docs.microsoft.com/en-us/azure/machine-learning/studio/algorithm-cheat-sheet)

39. Dentro de Machine Learning, Train, escoger el módulo "Train Model" y agregarlo al lienzo. Lo vamos a conectar Two-class boosted decission y con la primera salida de "Split Data". Como aparece en la imagen
![TitanicData](/images/Step39.png)
40. En "Launch column selector" de Train Model, vamos a seleccionar la columna "Survived" para indicar que es lo que queremos aprender.
41. Agregamos un módulo "Score Model" y "Evaluate Model" y los conectamos como en la siguiente imagen.
![TitanicData](/images/Step41.png)
42. Damos clic en Run.
43. Una vez que nuestro modelo se ha entrenado, damos clic en "Set Up Web Service" y seleccionamos la opción "Predictive Web Service [Recommended]"
![TitanicData](/images/Step43.png)
44. Así queda nuestro experimento como Web Service
![TitanicData](/images/Step44.png)
> Vamos a modificar un poco este experimento para poder enviar la información de regreso a SQL Server

45. Mover el módulo Select columns in dataset abajo de los módulos Edit Metadata como aparece en la imagen:
![TitanicData](/images/Step45.png)
46. Agregar otro módulo "Select Columns in Dataset" y conectarlo al último "Select Metadata"
![TitanicData](/images/Step46.png)
47. Agregamos un módulo "Add Columns" y lo conectamos como en la siguiente imagen
![TitanicData](/images/Step47.png)
48. Agregamos un módulo "Export Data" y lo conectamos a "Add Columns"
49. Configuramos el módulo "Export Data" para enviar la información a nuestra base de datos SQL Azure.
![TitanicData](/images/Step49.png)
50. Seleccionamos la opción "Accept any server certificate"
51. En comma separated list ponemos la siguiente lista (Las columnas de nuestro modelo): PassengerId, Name, Ticket, Cabin, Survived, PassengerClass, Gender, Age, SiblingSpouse, ParentChild, FarePrice, PortEmbarkation, Scored Probabilities
52. Poner el nombre de la tabla. Si estás siguiendo el ejemplo con los scripts, la tabla es "TitanicDataScored"
53. Comma Separated List (las columnas de la base de datos): PassengerId, Name, Ticket, Cabin, Survived, Pclass, Sex, Age, SibSp, Parch, Fare, Embarked, ScoredProbabilities
54. Seleccionar la opción "Allow writter success.."
![TitanicData](/images/Step54.png)
55. Dar clic en Run.
56. Dar clic en "Deploy web Service"

## Fase 3 . Consumir web Service ##
> Ahora que hemos configurado el experimento para que vuelva a escribir en SQL, podemos escribir nuestro script en R para llamar al servicio web de Azure ML. 
Azure ML realmente nos ayuda bastante aquí proporcionando código de muestra que podemos usar. 

57. En la etiqueta de servicios web, dar clic en "Request/Response" y navegar hasta la parte inferior de la página. Ahí tenemos el código R para comenzar.
![TitanicData](/images/Step57.png)

> Los siguientes pasos a seguir son:
 * Extraer la información de SQL que no le hemos dado una probabilidad
 * Llamar al servicio web de Azure ML para que nos haga la predicción en estos datos
> Para esto, necesitamos configurar un ODBC para podernos ocnectar a SQL desde R

58. Abrir la aplicación "ODBC Data Source Administrator"
![TitanicData](/images/Step58.png)

59. Dar clic en "Add" y escoger "ODBC Driver 11 for SQL"
60. Poner de nombre "SQLAzure" y poner la URL de nuestro servidor en Azure (<servername>.database.windows.net)
61. Dar clic en "Next" y seleccionar "With SQL Authentication". Poner las credenciales.
62. Dar clic en "Finish" y "Test Connectivity". Si las credenciales fueron correctas, te aparecerá un mensaje de éxito.
![TitanicData](/images/Step62.png)

> Ahora vamos a obtener las credenciales de nuestro web service para poder configurar el código de R
63. Regresar al experimento en AzureML y obtener el valor API Key
![TitanicData](/images/Step63.png)
64. En el paso 57 revisamos el código R que nos da AzureML, navegamos a la función curlPerform y obtenemos la URL.
![TitanicData](/images/Step65.png)
65. Abrimos RStudio
66. Creamos un nuevo RScript y colocamos el siguiente código
> Tenemos que modificar la conexión a nuestro SQL Server con nuestras credenciales, el API Key y la URL de nuestro servicio.

```{r}
library("RODBC")
library("RCurl")
library("rjson")


# This query simulates a table by generating a rowset with one integer column going from 1 to 1000
sqlQuery <- "SELECT * FROM dbo.TitanicData where PassengerId NOT IN (SELECT PassengerId FROM dbo.TitanicDataScored)"
#conn <- odbcDriverConnect(connectionString)

conn <- odbcConnect("SQLAzure", uid = "<tu username>", pwd = "<tu password>")

dataset <- sqlQuery(conn, sqlQuery)


close(conn)

if(nrow(dataset)>0)
{
  dataset <- dataset[,c(-1, -14)]
  dataset <- na.omit(dataset)
  
  createList <- function(dataset)
  {
    temp <- apply(dataset, 1, function(x) as.vector(paste(x, sep = "")))
    colnames(temp) <- NULL
    temp <- apply(temp, 2, function(x) as.list(x))
    return(temp)
  }
  
  # Accept SSL certificates issued by public Certificate Authorities
 
  options(RCurlOptions = list(cainfo = system.file("CurlSSL", "cacert.pem", package = "RCurl")))
  
  h = basicTextGatherer()
  hdr = basicHeaderGatherer()
  
  req =  list(
    Inputs = list(
      "input1"= list(
        list(
          'PassengerId' = "1",
          'Survived' = "1",
          'Pclass' = "1",
          'Name' = "",
          'Sex' = "",
          'Age' = "1",
          'SibSp' = "1",
          'Parch' = "1",
          'Ticket' = "",
          'Fare' = "1",
          'Cabin' = "",
          'Embarked' = ""
        )
      )
    ),
    GlobalParameters = setNames(fromJSON('{}'), character(0))
  )
  
  body = enc2utf8(toJSON(req))
  api_key = "" # El API Key de tu web service
  authz_hdr = paste('Bearer', api_key, sep=' ')
  
  h$reset()
  curlPerform(url = "https://ussouthcentral.services.azureml.net/workspaces/0f8b6eb5189c48d4b66e5b041b6317ba/services/e1d27adf48c24863aa051fd9a02e99ed/execute?api-version=2.0&format=swagger",
              httpheader=c('Content-Type' = "application/json", 'Authorization' = authz_hdr),
              postfields=body,
              writefunction = h$update,
              headerfunction = hdr$update,
              verbose = TRUE
  )
  
  headers = hdr$value()
  httpStatus = headers["status"]
  if (httpStatus >= 400)
  {
    print(paste("The request failed with status code:", httpStatus, sep=" "))
    
    # Print the headers - they include the requert ID and the timestamp, which are useful for debugging the failure
    print(headers)
  }
  
  print("Result:")
  result = h$value()
  print(fromJSON(result))
  
  finalResult <- fromJSON(result)
}
##Return results back
#inter <- do.call("rbind", finalResult$Results$output1$value$Values)
#titanicFinal <- data.frame(inter)
#names(titanicFinal) <- finalResult$Results$output1$value$ColumnNames
rm(list=setdiff(ls(), "dataset"))

conn <- odbcConnect("SQLAzure", uid = "tu user", pwd = "tu password")
dataset <- data.frame(sqlQuery(conn, "SELECT * FROM dbo.TitanicDataScored")) 
close(conn)

```
67. Dar clic en Run para validar cada uno de los pasos en el script. Una vez que el script esté listo y configurado, vamos a hacer este proceso desde PowerBI.
68. Abrir PowerBI y seleccionar como fuente de datos "RScript"
![TitanicData](/images/Step68.png)
69. Pega el script de R que configuraste en los pasos anteriores.
70. Ya cuentas con la información para poder comenzar a hacer tu tablero! Intenta generar algunas gráficas. Este es el tablero que yo creé.
![TitanicData](/images/Step70.png)
