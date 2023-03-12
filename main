from multiprocessing import Process
from multiprocessing import BoundedSemaphore, Semaphore, Lock
from multiprocessing import current_process
from multiprocessing import Value, Array
from time import sleep
import random



N = 10              #Cuantos numeros generan
Buffer_size = 3     #Tamaño de las listas
NPROD = 3           #Numero de productores




#Añade un elemento a la lista
def add_data(array, data, index, cota, non_empty):
    try:
        cota.acquire() 
        array[index] = data 
        non_empty.release()   
        delay(6)
    finally:  
        pass

#Cambia un elemento de la lista a vacio (-2)        
def remove_data(array,index,cota,non_empty):
    
    try: 
        cota.release()
        non_empty.acquire()
        array[index]=-2
    finally:
        pass
    

        

#Devuelve el mínimo de una lista y su posición en ella        
def min_indice(lst):
    minimo = min(lst)
    indice = lst.index(minimo)    
    return (indice, minimo)
  

def delay(factor = 3):
    sleep(random.random()/factor)


#proceso que produce números no negativos de forma creciente y los añade a una lista
def producer(array, cota, non_empty):
    data=0
    index=0
    for _ in range(N):
        

        data = data + random.randint(0,5)
        add_data(array, data, index, cota, non_empty)
        index= (index + 1)%Buffer_size
        print (f"producer {current_process().name} produzco {data}")
        
    data= -1
    add_data(array, data, index, cota, non_empty)
    index= (index+1) % Buffer_size
    

#Proceso que espera a que haya por lo menos un elemento cada lista de los productores
# y entonces introduce el mínimo de esos numeros en una lista ordenada.
def consumer_merge(storage, cota_lst, non_empty_lst):
    ordenada=[]
    primer_elem=[None]*NPROD
    index_cons = [0]*NPROD
   
    while True:
        #Comprueba que haya algún elemento en cada lista de los productores
        for non_empty in non_empty_lst:
            non_empty.acquire()
            non_empty.release()
        
        c=0
        #Llena una lista con los primeros elementos producidos de cada productor
        for array in storage:
            indice=index_cons[c]
            elemento=array[indice]
            
            #Los -1 implican que el productor terminó por lo que los cambia por infinito
            if elemento != -1:
                primer_elem[c]=elemento
            
            else:                           
                primer_elem[c]=float('inf') 
            c+=1
        
        #Si todos los productores terminaron detiene el consumer
        if primer_elem == [float('inf') for _ in range(NPROD)]:
            break    

        (i,minimo)=min_indice(primer_elem)
        
        remove_data(storage[i], index_cons[i] , cota_lst[i],non_empty_lst[i])
        
        index_cons[i]=(index_cons[i]+1) % Buffer_size
        
        ordenada.append(minimo)
        
        
        print (f"consumer {current_process().name} ordenando {ordenada}")
        delay()

        
#Inicializa los procesos, los semaforos y el storage. 
def main():
  
    
    storage=[Array("i",Buffer_size) for _ in range(NPROD)]
    
    
    for array in storage:
        for i in range(Buffer_size):
            array[i]=-2
        
    #Se asegura de que haya algún elemento
    non_empty_lst= [Semaphore(0) for _ in range(NPROD)]

    #Se asegura de que haya espacio para que el productor produzca más números
    cota_lst= [BoundedSemaphore(Buffer_size) for _ in range(NPROD)]

    

    prodlst = [ Process(target=producer,
                        name=f'prod_{i}',
                        args=(storage[i], cota_lst[i], non_empty_lst[i]))
                for i in range(NPROD) ]

    conslst =  Process(target=consumer_merge,
                      name="cons",
                      args=(storage, cota_lst, non_empty_lst))
    
    for p in prodlst + [conslst]:
        p.start()
    
    for p in prodlst + [conslst]:
        p.join()


if __name__ == '__main__':
    main()
