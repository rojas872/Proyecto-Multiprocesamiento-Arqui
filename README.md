# Proyecto-Multiprocesamiento-Arqui
#Se crea un programa con el cual se pueda reconocer rostros, accesorios y entre otras mas cosas desde videos cortos, peliculas etc.
#El sistema registra los videos por medio de imagenes que va sacando de los videos.


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

#Se usa una direccion url y una suscipcion en microsoft azure, para poder hacer uso del reconocimiento de personas

       subscription_key = None
       SUBSCRIPTION_KEY = '4a627e3e7d46478fa2b14c2f41f7f91e'
       BASE_URL = 'https://tallerp2021.cognitiveservices.azure.com/face/v1.0/'
       CF.BaseUrl.set(BASE_URL)
       CF.Key.set(SUBSCRIPTION_KEY)


       listaPath=[] 


       def clearConsole():
              command = 'clear'
              if os.name in ('nt', 'dos'):  # If Machine is running on Windows, use cls
                     command = 'cls'
              os.system(command)


       def fotogramas(pathVideo):

       #C:\\Users\DELL 7470\\OneDrive\\Escritorio\\Fotograma\\opencv_extract_frames\\Kevelyn_Gonzalez_GR51.mp4
              # Extract all frames from a video using Python (OpenCV)
              cap = cv2.VideoCapture(pathVideo)

              img_index = 0
              j= 20
              while (cap.isOpened()):
                     ret, frame = cap.read() #obtiene cada fotograma
                     if ret == False: 
                            break
                     if img_index==j: #obtiene el fotograma cada 20
                           cv2.imwrite('C:\\Users\Andrey Rojas Garcia\\Desktop\\Fotograma' + str(img_index) + '.png',frame)
                           foto= "C:\\Users\Andrey Rojas Garcia\\Desktop\\Fotograma" + str(img_index) + '.png'
                           listaPath.append(foto)
                           j +=20           
                     img_index += 1       
              cap.release()
              cv2.destroyAllWindows() #destruye ventanas que se pudieron generar

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

       #def mide_tiempo(funcion):
              # def funcion_medida(*args, **kwargs):
                     # inicio = time.time()
                     # c = funcion(*args, **kwargs)
              #        print(time.time() - inicio)
              #        return c
              # return funcion_medida

       #@mide_tiempo

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
              cont= 1;

              for x in analysis:
                     print("\nPersona # ", cont)
                     lista=[]
                     faceId= x["faceId"]
                     personas.append(faceId)
                     faceAttributes= x['faceAttributes']
                     emotion = faceAttributes["emotion"]
                     gender= faceAttributes["gender"]
                     accessories= faceAttributes['accessories']
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

       if __name__ == "__main__":
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

              #https://sample-videos.com/video123/mp4/720/big_buck_bunny_720p_1mb.mp4
              #C:\Users\DELL 7470\Downloads\video.mp4
              #C:\Users\Andrey Rojas Garcia\Desktop\Mr Robot\Temporada 1\1x01.mp4
