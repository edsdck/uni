---
tags: [VGTU]
title: Lygiagretusis programavimas
created: '2020-05-09T07:59:24.612Z'
modified: '2020-05-09T10:36:32.510Z'
---

# Lygiagretusis programavimas

> Gijų programavimas. Pateikite ir pademonstruokite gijų programavimo pagrindinius principus. Kaip kuriamos gijos? Kaip jos sąveikauja? Kokie kintamieji yra bendri, o kokie lokalus? Kaip perduoti duomenis gijai? Pateikti kodo pavyzdį.

Gijos yra sukuriamos iš master gijos naudojantis komanda `pthread_create (thread,attr,start_routine,arg)`
Argumentai
* thread - gijos indenfikatorius
* attr - įvairūs atributai (jeigu NULL bus panaudota default)
* start_routine - funkcija, kuri bus įvykdyta gijoje
* arg - perduodamas argumentas į funkciją

Bendri duomenys yra visi globalūs kintamieji ir static kintamieji, taip pat tai gali būti dinaminiai duomenys sukurti ant heap (tačiau reikės perduoti atitinkamą rodyklę). Visi kintamieji apibrėžiami funkcijoje yra lokalūs, tai yra prieinami tik tai gijai. 

Norint perduoti duomenis į giją geriausia naudotis sukuriant duomenų struktūra ar klasę. Duomenys į giją yra perduodami gijos kūrimo metu kaip argumentas `arg` viršuje aprašytoje funkcijoje.

```cpp
#include <pthread.h>
#include <iostream>
#include <cstdlib>
#include <string>

using namespace std;

class thread_data
{
public:
  string x;
};

void *SomeFunction(void *data)
{
  thread_data *myData = reinterpret_cast<thread_data *>(data);
  std::cout << myData->x << endl;
 
  delete myData;

  return NULL;
}
 
int main()
{
  pthread_t thread;
  thread_data *data = new thread_data();
  data->x = "Hello world";
 
  pthread_create(&thread, NULL, SomeFunction, data);
 
  pthread_join(thread, NULL);
  
  return 0;
}
```
---
> Gijų programavimas. Gijų darbo sinchronizavimas. Kas yra barjeras? Pateikite kodo pavyzdį ir paaiškinkite jo veikimą. Kas yra užraktas Pthread mutex? Pateikite kodo pavyzdį ir paaiškinkite jo veikimą.

Barjeras tai yra lyg siena, kuri nepraleidžia gijos eiti toliau iki kol visos aprašytos gijos neateina iki šio barjero.
```cpp
#include <pthread.h>
#include <iostream>
#include <cstdlib>

using namespace std;

pthread_barrier_t barrier;

class thread_data
{
public:
  int id;
};

void *PrintHello(void *passedThreadData)
{
   thread_data *myData = reinterpret_cast<thread_data *>(passedThreadData);
   
   pthread_barrier_wait(&barrier);

   cout << "I'm thread #" << myData->id <<endl;

   delete myData;
   return NULL;
}

int main (int argc, char *argv[])
{
   const int NUM_THREADS = 4;
   pthread_barrier_init (&barrier, NULL, NUM_THREADS);
   pthread_t threads[NUM_THREADS];

   for(int t=0; t<NUM_THREADS; t++){
      thread_data *data = new thread_data();
      data->id=t;
      int rc = pthread_create(&threads[t], NULL, PrintHello, data);
      if (rc){
         cerr << "ERROR; return code from pthread_create() is " << rc << endl;
         exit(-1);
      }
   }

   for(int t=0; t<NUM_THREADS; t++)
      pthread_join(threads[t], NULL);

   pthread_barrier_destroy(&barrier);

   return 0;
}
```
Mutex yra užraktas, kurį panaudojus gija užsirakina ir kitos gijos pasiekusios tą patį užraktą užsiblokuoja ir laukia, kol užsirakinusi gija baigs darba ir atrakins mutex.

```cpp
#include <pthread.h>
#include <iostream>
#include <cstdlib>

using namespace std;

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

class thread_data
{
public:
  int id;
};

void *PrintHello(void *passedThreadData)
{
   thread_data *myData = reinterpret_cast<thread_data *>(passedThreadData);

   pthread_mutex_lock(&mutex);
   cout << "Hello World! It's me, thread #" << myData->id <<endl;
   pthread_mutex_unlock(&mutex);
   cout << "mutex unlocked by #" << myData->id <<endl;

   delete myData;
   return NULL;
}


int main (int argc, char *argv[])
{
   const int NUM_THREADS = 4;
   pthread_t threads[NUM_THREADS];

   pthread_mutex_init(&mutex, NULL);

   for(int t=0; t<NUM_THREADS; t++){
      thread_data *data = new thread_data();
      data->id=t;
      int rc = pthread_create(&threads[t], NULL, PrintHello, data);
      if (rc){
         cerr << "ERROR; return code from pthread_create() is " << rc << endl;
         exit(-1);
      }
   }
   
   for(int t=0; t<NUM_THREADS; t++)
      pthread_join(threads[t], NULL);

   pthread_mutex_destroy(&mutex);

   return 0;
}
```
---
> Gijų programavimas. Gijų darbo sinchronizavimas. Pateikite ir paaiškinkite deadlock susidarymo pavyzdį.

```cpp
#include <pthread.h>
#include <iostream>
#include <cstdlib>

using namespace std;

pthread_mutex_t mutex_1 = PTHREAD_MUTEX_INITIALIZER;
pthread_mutex_t mutex_2 = PTHREAD_MUTEX_INITIALIZER;

void *functionOne(void *data)
{
   pthread_mutex_lock(&mutex_1);
   pthread_mutex_lock(&mutex_2);
   
   cout << "Calling from function 1" << endl;
   
   pthread_mutex_unlock(&mutex_2);
   pthread_mutex_unlock(&mutex_1);

   return NULL;
}

void *functionTwo(void *data)
{
   pthread_mutex_lock(&mutex_2);
   pthread_mutex_lock(&mutex_1);
   
   cout << "Calling from function 2" << endl;
   
   pthread_mutex_unlock(&mutex_1);
   pthread_mutex_unlock(&mutex_2);

   return NULL;
}

int main (int argc, char *argv[])
{
   pthread_t threads[2];
   
   pthread_mutex_init(&mutex_1, NULL);
   pthread_mutex_init(&mutex_2, NULL);

   pthread_create(&threads[0], NULL, functionOne, NULL);
   pthread_create(&threads[1], NULL, functionTwo, NULL);

   pthread_mutex_destroy(&mutex_1);
   pthread_mutex_destroy(&mutex_2);

   return 0;
}
```
---
> Gijų programavimas. Lygiagrečiai apskaičiuoti vektoriaus elementų sumą naudojant gijas.
```cpp
#include <stdio.h>
#include <pthread.h>
#define array_size 1000
#define no_threads 10

float a[array_size];
int global_index = 0;
int sum = 0;
pthread_mutex_t mutex1;

void *slave(void *ignored)
{
    int local_index, partial_sum = 0;
    do {
        pthread_mutex_lock(&mutex1);
        local_index = global_index;
        global_index++;
        pthread_mutex_unlock(&mutex1);

        if (local_index < array_size)
            partial_sum += *(a + local_index);
    }
    while(local_index < array_size);

    pthread_mutex_lock(&mutex1);
    sum += partial_sum;
    pthread_mutex_unlock(&mutex1);

    return 0;
}

int main()
{
    int i;
    pthread_t thread_x[10];
    pthread_mutex_init(&mutex1, NULL);

    for (i = 0; i < array_size; i++)
        a[i] = i+1;

    for (i = 0; i < no_threads; i++)
        pthread_create(&thread_x[i] , NULL, slave,NULL);

    for (i = 0; i < no_threads; i++)
        pthread_join(thread_x[i] , NULL);

    printf("The sum of 1 to %d is %d\n", array_size, sum);
}
```
---
> OpenMP duomenų tipai (shared, private, ...). Pateikite OpenMP duomenų tipų nustatymo taisykles. Pavyzdžių pagalba paaiškinkite, kada reikia naudoti vieną ar kitą tipą.
### Implicit
Bendri (shared)
* Pagal nutylėjimą (default) iki lygiagrečiosios srities pradžios apibrėžti kintamieji srities viduje.
* Globalūs ir static kintamieji.

Lokalūs (private)
* Ciklų indeksų kintamieji.
* Steko kintamieji sukurti funkcijoje iškviestoje paraleliai.
### Explicit
**Shared** tipas skirtas aprašyti kintamuosius, kurie būtų bendri tarp visų darbo gijų. 
OpenMP neapsaugo nuo data races, tuo turi pasirūpinti programuotojas. Be to, bendri kintamieji sukuria overhead, taigi reikia stengtis naudoti kuo mažiau.
```cpp
#pragma omp parallel for shared(n, a)
for (int i = 0; i < n; i++)
{
    int b = a + i;
    ...        
}
```
**Private** tipas aprašo kintamuosius, kurie yra kiekvienai gijai atskiri, taigi tą kintamajį gali pasiekti tik ta gija. Tai reiškia, kad kintamasis yra atkartojamas ir sukuriamas naujai kiekvienai gijai. Reikėtų prisiminti, tai, kad jei kintamasis ir buvo apibrėžtas prieš paralelų regioną, jis nebebus apibrėžtas pradėjus darbą gijose. 
```cpp
int p = 10; 
#pragma omp parallel private(p)
{
    // the value of p is undefined
    p = omp_get_thread_num();
    // the value of p is defined
}
```
**default(shared)** visi naudojami kintamieji paraleliam regione yra bendri.

**default(none)** priverčiama aprašyti visų kintamujų bendrinimo atributus. Jeigu pamirštama aprašyti - kompiliatorius tai primena *error* žinute.
```cpp
int n = 10;
std::vector<int> vector(n);
int a = 10;

#pragma omp parallel for default(none) shared(n, vector, a)
for (int i = 0; i < n; i++) 
{
    vector[i] = i * a;
}
```
---
> OpenMP užduočių paskirstymas. Pateikite OpenMP užduočių paskirstymo konstrukcijas ir paaiškinkite jų veikimą pavyzdžių pagalba.

1. `#pragma omp for`
```cpp
#include <omp.h>
 #define N 1000
 #define CHUNKSIZE 100

 main(int argc, char *argv[]) {

 int i, chunk;
 float a[N], b[N], c[N];

 /* Some initializations */
 for (i=0; i < N; i++)
   a[i] = b[i] = i * 1.0;
 chunk = CHUNKSIZE;

 #pragma omp parallel shared(a,b,c,chunk) private(i)
   {
    #pragma omp for schedule(dynamic,chunk) nowait
    for (i=0; i < N; i++)
      c[i] = a[i] + b[i];

   }   /* end of parallel region */
 }
```
Galimos ir šios konstrukcijos:
* `#pragma omp for schedule(static)`
Visos iteracijos yra padalinamos po lygiai, jeigu įmanoma.
* `#pragma omp for schedule(guided)`
Panašiai kaip `dynamic`, tačiau gijai paprašius naujo duomenų `chunk` jo dydis vis mažėja.
* `#pragma omp for schedule(runtime)`
Programos vykdymo metų tvarkaraščio tipas (schedule) ir bloko dydis (chunk) yra paimami iš aplinkos kintamojo OMP_SCHEDULE (environment variable).

2. `#pragma omp sections`

Lygiagrečiąją sritį vykdančios gijos lygiagrečiai atlieka skirtingas
sekcijas (struktūrinius blokus). 
Kiekviena sekcija bus atlikta tik vieną kartą vienos iš grupės gijų.
```cpp
#include <iostream>

using namespace std;

void function_1()
{
    for (int i = 0; i != 3; i++)
    {
        cout << "Function 1 (i = " << i << ")" << endl;
        sleep(1);
    }
}

void function_2()
{
    for (int j = 0; j != 4; j++)
    {
        sleep(1);
        cout << "                   Function 2 "
                  << "(j = " << j << ")" << endl;
    }
}

int main( int argc, char* argv[] )
{
	#pragma omp parallel sections
    {    
        #pragma omp section
        function_1();
            
        #pragma omp section
        function_2();
    }
  return 0;
}

```
3. `#pragma omp single`

Jei lygiagrečioje srityje reikia nurodyti struktūrinį bloką, kuris būtų įvykdytas tik vieną kartą, t.y. tik vienos (nesvarbu kokios) gijos, tai galime padaryti su single direktyva. 

Pirmoji gija, kuri vykdydama lygiagrečiąją sritį pasieks šią direktyvą, imsis vykdyti nurodytą struktūrinį bloką, o kitos gijos jį praleis ir lauks konstrukcijos pabaigoje (jei nenurodytas argumentas nowait).
```cpp
#include <iostream>
#include <cstdio>
#include "omp.h"

using namespace std;

int main( int argc, char* argv[] )
{
  #pragma omp parallel
  {
    int id = omp_get_thread_num();
    //cout << "Hello, world, from thread - " << id << endl;
    printf("Hello, world, from thread - %d\n", id);
  
    #pragma omp single
    {
      //int nthreads = omp_get_num_threads();
      //cout << id << ": Number of threads = " << nthreads << endl;
      printf("Single block performed by thread - %d\n", id);
    }
  }

  return 0;
}

```
