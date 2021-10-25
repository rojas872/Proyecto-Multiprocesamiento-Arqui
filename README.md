# Proyecto-Multiprocesamiento-Arqui
Hola, en esta ocasion la idea de crear este proyecto es para poder analizar datos con una eficiencia mejor en cuanto al tiempo de que
se dura en recibir los datos y a la vez de generarlos. Es por esto que:

Se crea un programa en donde permita poder obtener reconocimiento de personas, ademas de sus accesorios, color de cabello, entre otros mas
que pueden estar presentes durante el video, eso si debe ser parte de las personas y con previa busqueda que se realiza de algunas ocasiones.

#Se importan algunas librerias y modulos para trabajar con algunos servicios de Microsof Azure y opencv para reconocer emociones en videos

#Paso 1.    presionar windows + R

#Paso 2.    escribir "powershell" en la pequena ventana que se muestra

#Paso 3.    escribir "pip" en la terminal que se muestra, si no hay instalado ningun paquete pip precedemos a ingresar los siguientes comandos para sun instalacion

#Paso 4.    instalaremos el modulo pygame escribimos pip install pygame 

#Paso 5.     instalaremos opencv esta libreria permite el analisis y procesamiento de videos e imagenes escribimos pip install opencv-python

#Paso 6.     Actualizamos los paquetes pip existentes python -m pip install --upgrade pip



              from multiprocessing import Process
              import multiprocessing
              import sys
              import requests
              import json 
              import os

              import cognitive_face as CF
              from PIL import Image, ImageDraw, ImageFont
              import cv2



              import concurrent.futures
              import time

              subscription_key = None
              #Se requiere de una suscripcion  Microsof azure para obetener un URl y una KEY
              SUBSCRIPTION_KEY = '4a627e3e7d46478fa2b14c2f41f7f91e'
              BASE_URL = 'https://tallerp2021.cognitiveservices.azure.com/face/v1.0/'
              CF.BaseUrl.set(BASE_URL)
              CF.Key.set(SUBSCRIPTION_KEY)


#Se guardadn las rutas de acceso de las imagenes en una lista
listaPath=[] 

"""[Este metodo permite limpiar la terminal]
"""
              def clearConsole():
                     command = 'clear'
                     if os.name in ('nt', 'dos'):  
                            command = 'cls'
                     os.system(command)

"""[como se menciona anteriromente usamos la libreria opencv, para analizar el video, recibe por parametro el path local del video, capturando el fotograma cada 20]
"""
              
              def fotogramas(pathVideo):
                     # Extract all frames from a video using Python (OpenCV)

                     cap = cv2.VideoCapture(pathVideo)

                     img_index = 0
                     j= 20
                     while (cap.isOpened()):
                            ret, frame = cap.read() #obtiene cada fotograma
                            if ret == False: 
                                   break
                            if img_index==j: #obtiene el fotograma cada 20
                                  cv2.imwrite('C:\\Users\\DELL 7470\\OneDrive\\Escritorio\\Fotograma\\opencv_extract_frames' + str(img_index) + '.png',frame)
                                  foto= "C:\\Users\\DELL 7470\\OneDrive\\Escritorio\\Fotograma\\opencv_extract_frames" + str(img_index) + '.png'
                                  listaPath.append(foto)
                                  j +=20           
                            img_index += 1       
                     cap.release()
                     cv2.destroyAllWindows()#destruye ventanas que se pudieron generar

#En esta funcion se muestra la imagen usada y a su vez con un rectangulo en el rostro de la persona o las personas que reconozca 
              
              def show_image(path, faces):
                     """ Show the picture with rectangle in the faces
                     Arguments:
                     faces {list} -- list of faceRectangle values
                     path {string} -- image location 
                     """
                     image = Image.open(path)
                     draw = ImageDraw.Draw(image)
                     for faceRectangle in faces:
                            top = faceRectangle['top']
                            left = faceRectangle['left']
                            width= faceRectangle['width']
                            height = faceRectangle['height']
                            draw.rectangle((left,top, left+width, top+height), outline='red', width=2)

                     image.show()

              def higher(lista):
                     """ extracts the highest data from the list
                     Arguments:
                     lista {list}-- list with sublist that containing the faceId, age and gender 

                     Return:
                     higher -- is the sublist with the highest age

                     """
                     if len(lista)==0:
                            return None 
                     else:
                            higher= lista[0]
                            for x in lista[1:]:
                                   if x[1]>higher[1]:
                                          higher=x
                            return higher 


"""[En esta parte, lo que sucede es que se toma las imagenes que se desean usar en la busqueda de emociones y demas datos que se deseen
       tomar, ya sea como sus emociones, color de cabello, accesorios, entre otros mas.]
"""
              
              def emotions(picture):
                     """ use people's information list 

                     Arguments :

                     picture {string} -- is the image 

                     Return:
                     a list with information about people  
                     """
                     image_path = picture

                     image_data = open(image_path, "rb").read()
                     headers = {'Ocp-Apim-Subscription-Key': SUBSCRIPTION_KEY,
                     'Content-Type': 'application/octet-stream'}
                     params = {
                            'returnFaceId': 'true',
                            'returnFaceLandmarks': 'false',
                            'returnFaceAttributes': 'age,gender,headPose,smile,facialHair,glasses,emotion,hair,makeup,occlusion,accessories,blur,exposure,noise',
                     }
                     response = requests.post(
                                                 BASE_URL + "detect/", headers=headers, params=params, data=image_data)
                     analysis = response.json()
                     #print (analysis)
                     personas=[]
                     cont= 1

      
  Se recorren las listas para ir sacando cada uno de los datos requeridos incluidos en diccionarios, de estos datos se encuentran
  las emociones, el color de cabello de la persona, los accesorios con los que ande puesto, entre otros mas.
  Para poder acceder a cada uno de las areas de los diccionarios, lo que se hace es ir recorriendo el diccionario, y por consiguiente
  sacando uno a uno de los datos, como es en el caso de las emociones, se situa primero sobre el faceAttributes para despues buscar dentro
  de el y sacar las emociones, asi como el genero y hasta los accesorios, se basa en una misma accion para poder obtener dichos datos.
  Luego de que estos datos se sacan se incluyen en listas y luego esta lista se agrega a otra lista para que sean varias sublistas en con datos.
            
       
       
              for x in analysis:
                     print("\nPersona # ", cont)
                     lista=[]
                     faceId= x["faceId"]                         #Se accede al face id
                     personas.append(faceId)                     #Se agrega el faceID a la lista de personas
                     faceAttributes= x['faceAttributes']         #Se accede a faceAttributes
                     emotion = faceAttributes["emotion"]         #Luego se accede a emotion desde faceaAttributes
                     gender= faceAttributes["gender"]            #Tambien se accede a gender desde faceAttributes
                     accessories= faceAttributes['accessories']  #Y asi con los datos que se quieran seguir sacando de acuerdo al diccionario que se genera
                     facialHair= faceAttributes['facialHair']
                     hair= faceAttributes['hair']
                     hairColor= hair ['hairColor']

                     #imprime el color de cabello
                     print("\nHair color: ")
                     for x in hairColor:   
                            color= x["color"]
                            confidence= x["confidence"]
                            lista1=[]
                            lista1.append(color)
                            lista1.append(confidence)
                            lista1.append(faceId)
                            lista.append(lista1) 
                     if lista==[]:
                            lista.append(faceId)
                            print("faceId: ", faceId, " doesn't hair ")
                     else: 
                            p= higher(lista)        
                            print("\nfaceId: ", p[2]," predominant hair color: ", p[0])
                     print("\nGender: ")
                     print(gender)
                     # imprime bello facial
                     if gender=="male": 
                            print("\nFacial features: ")  
                            print (facialHair) 
                     #Imprime las emociones
                     print("\nEmotion: ")
                     print(emotion)
                     #imprime los accesorios      

                     if accessories==[]:
                            print("\nAccessories: ")
                            print("\nthe person don't have accessories")
                     else:
                            print('\nAccessories:')
                            dic= accessories
                            for x in dic:
                                   tipo= x['type']
                                   print(tipo)
                     cont=cont+1
              faces = []
              #Saca el rectangulo en la cara de la persona
              for face in analysis:
                     faceRectangle = face['faceRectangle']
                     faces.append(faceRectangle)
              show_image(picture,faces)
              print("\n\nNumber of people in the image:  ", len(personas))

Se realiza un menu con la idea de poder mostrar ambos tiempos, tanto en el que no se usa el multiprocesamiento y en el que se hace 
uso de ello, de lo cual denota que es bastante mas eficiente que sin usarse.

       if _name_ == "_main_":
              print ("\nSTARTING...")
              a = 0
              while a!='3':
                     print ("1. Funcion sin multiprocesamiento")
                     print ("2. Funcion con multiprocesamiento")
                     print("3. Salir")
                     a = input("\nEnter the number:")
                     if a == '1':
                            #Se pide el path del video                  
                            pathVideo=input("\nEnter the path of the video:")
                            start = time.perf_counter()
                            #se analiza el video
                            fotogramas(pathVideo)
                            #parte de Azure
                            for x in listaPath:
                                   print(x)
                                   emotions(x) 
                                   #os.system("pause")
                                   clearConsole()
                            print ("\nSin multiprocesamiento")
                            print(f'Duracion: {time.perf_counter() - start}')
                            listaPath=[]

En esta parte lo que se realiza es el uso del multiprocesamiento por parte del recurso face y del fotograma, y se sacan los tiempos de ejecucion.

                      #MULTIPROCESAMIENTO DEL RECURSO FACE       
                     elif a =='2':

                            pathVideo=input("\nEnter the path of the video:")
                            start = time.perf_counter()
                            p= multiprocessing.Process(target= fotogramas,args=(pathVideo,))
                            p.start()
                            p.join()
                            #parte de Azure
                            fotogramas(pathVideo)
                            #Funcion con Multiproceso
                            start = time.perf_counter()

                            with concurrent.futures.ProcessPoolExecutor() as executor:
                                   for x in listaPath:

                                          executor.submit(emotions(x))            
                            print ("\nCon multiprocesamiento")
                            print(f'Duration: {time.perf_counter() - start}')

                            print("\\n")  
                            listaPath=[]

              print ("\nEND")
