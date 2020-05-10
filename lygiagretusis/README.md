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
---
> OpenMP užduočių vykdymo sinchronizavimo konstrukcijos. Pateikite OpenMP užduočių vykdymo sinchronizavimo konstrukcijas ir paaiškinkite jų veikimą pavyzdžių pagalba.
1. `#pragma omp barrier`

Kai gija pasiekia šią direktyvą, ji sustoja ir laukia, kol visos kitos šios lygiagrečiosios srities gijos pasieks šį barjerą.

OpenMP pats įstato barjerus užduočių paskirstymo (for, sections, single) konstrukcijų pabaigoje, jei nenurodytas argumentas nowait.
```cpp
void work1(int k) {
  // large amount of work
}
void work2(int k) {
  // large amount of work that must all happen after work1 is finished
}

int main() {
  int n=1000000;
  #pragma omp parallel private(i) shared(n) {
    #pragma omp for
    for (i=0; i<n; i++)
      work1(i);

    #pragma omp barrier
    #pragma omp for
    for (i=0; i<n; i++)
      work2(i);
  }

  return 0;
}
```
2. `#pragma omp critical`

Kritinės sekcijos struktūrinis blokas vienu metu gali būti vykdomas tik vienos gijos.

```cpp
#include <iostream>
#include <omp.h>

using namespace std;

int main()
{
  int x = 0;
  cout << "Initial x = " << x << endl;

  #pragma omp parallel num_threads(4)
  {
    #pragma omp critical 
    for (int i=0; i < 1000; i++)
      x = x + 1;

  } 

  cout << "Final x = " << x << endl;
}
```
3. `#pragma omp atomic`

Jei reikia tik išvengti “race condition” vienam kintamajam.
```cpp
#pragma omp parallel
{
  int mytid = omp_get_thread_num();
  double tmp = some_function(mytid);
  #pragma omp critical
  sum += tmp;
}
```

4. `#pragma omp master`

Direktyva nurodo, kad toliau sekantis struktūrinis blokas yra vykdomas tik pagrindinės gijos (master thread), o visos kitos tos grupės gijos tą bloką praleidžia.

```cpp
#pragma omp parallel
{
  int id = omp_get_thread_num();
  some_work(id);
  #pragma omp master
  {
    some_work_master();
  }
}
```
---
> Pagrindinės OpenMP bibliotekos funkcijos. Paaiškinkite funkcionalumą ir pateikite taikymo pavyzdžius.

```cpp
omp_set_num_threads(int num_threads)
```
Sets the number of threads that will be used in the next parallel region.

```cpp
omp_get_num_threads()
```
Returns the number of threads that are currently in the team executing the parallel region from which it is called.

```cpp
omp_get_thread_num(void)
```
Returns the thread number of the thread, within the team, making this call. This number will be between 0 and OMP_GET_NUM_THREADS-1. The master thread of the team is thread 0.
```cpp
omp_get_wtime()

double start = omp_get_wtime();
// some work
double end = omp_get_wtime();
cout << “Darbas uztruko “ << (end - start) << “sekundziu“ << endl;
```
Provides a portable wall clock timing routine.

[Likusios funkcijos](https://computing.llnl.gov/tutorials/openMP/#RunTimeLibrary)

---
> Pateikite OpenMP kodą matricos ir vektoriaus sandaugos apskaičiavimui ir paaiškinkite jo vykdymą.

```cpp
#include <iostream>
#include <omp.h>

using namespace std;

void runCalculation(int size, int threads);

int main() {
    runCalculation(2000, 8);
}

void runCalculation(int size, int threads) {
    double *B = new double[size];
    double *C = new double[size];
    double **A = new double *[size];

    for (int i = 0; i < size; i++)
        A[i] = new double[size];

    for (int i = 0; i < size; i++)
        for (int j = 0; j < size; j++)
            A[i][j] = 300 + i + i * j;
    for (int j = 0; j < size; j++)
        B[j] = 50 + 3 * j;

    double t1, t2;

    t1 = omp_get_wtime();
    omp_set_num_threads(threads);
    {
      for (int i = 0; i < size; i++)
        C[i] = 0;
      #pragma omp parallel for
      for (int i = 0; i < size; i++)
        for (int j = 0; j < size; j++)
          C[i] += A[i][j] * B[j];
    }
    t2 = omp_get_wtime();

    double time = (double) (t2 - t1);

    for (int i = 0; i < size; i++)
        delete[]A[i];
    delete[]A;
    delete[]B;
    delete[]C;
}
```

> Pateikite dvi standartines MPI duomenų persiuntimo funkcijas. Aprašykite ir aptarkite jų argumentus. Pateikite duomenų persiuntimo su MPI pavyzdį.

```cpp
int MPI_Send(void *buf, int count, MPI_Datatype datatype, 
  int dest, int tag, MPI_Comm comm);
```
* buf – buferio, kuriame laikomi siunčiami duomenys, pradžios adresas (rodyklė)
* count – siunčiamų duomenų elementų kiekis (skaičius)
* datatype – siunčiamų duomenų elementų tipas (MPI tipas)
* dest – proceso, kuriam siunčiamas šis pranešimas (t.y. gavėjo), numeris (rank’as) komunikatoriuje comm
* tag – šiam pranešimui programuotojo suteikiamas numeris (paprastai, kad butų galima šį pranešimą atskirti nuo kitų, bet jis (tag) nebūtinai turi būti unikalus, t.y. gali būti ir vienodas visiems siunčiamiems pranešimams)
* comm – komunikatorius, kuriam priklauso abu procesai (ir siuntėjas, ir gavėjas).

```cpp
MPI_Recv(void *buf, int count, MPI_Datatype datatype,
 int source, int tag, MPI_Comm comm, MPI_Status *status);
```
* buf – buferio, į kurį bus patalpinti atsiusti duomenys, pradžios adresas (rodyklė)
* count – siunčiamų duomenų elementų kiekis (skaičius)
* datatype – siunčiamų duomenų elementų tipas (MPI tipas)
* source – proceso, iš kurio turi būti gautas šis pranešimas (t.y. siuntėjo), numeris (rank’as) komunikatoriuje comm
* tag – šiam pranešimui programuotojo suteikiamas numeris (paprastai, kad butų galima šį pranešimą atskirti nuo kitų, bet jis (tag) nebūtinai turi būti unikalus, t.y. gali būti ir vienodas visiems siunčiamiems pranešimams)
* comm – komunikatorius, kuriam priklauso abu procesai (ir siuntėjas, ir gavėjas).
* status – rodyklė į MPI duomenų struktūrą, į kurią bus įrašyti įvykusios duomenų gavimo operacijos duomenys (source, tag, message size)

```cpp
#include <mpi.h>
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char** argv) {
  // Initialize the MPI environment
  MPI_Init(NULL, NULL);
  // Find out rank, size
  int world_rank;
  MPI_Comm_rank(MPI_COMM_WORLD, &world_rank);
  int world_size;
  MPI_Comm_size(MPI_COMM_WORLD, &world_size);

  int number;
  if (world_rank == 0) {
    // If we are rank 0, set the number to -1 and send it to process 1
    number = -1;
    MPI_Send(
      /* data         = */ &number, 
      /* count        = */ 1, 
      /* datatype     = */ MPI_INT, 
      /* destination  = */ 1, 
      /* tag          = */ 0, 
      /* communicator = */ MPI_COMM_WORLD);
  } else if (world_rank == 1) {
    MPI_Recv(
      /* data         = */ &number, 
      /* count        = */ 1, 
      /* datatype     = */ MPI_INT, 
      /* source       = */ 0, 
      /* tag          = */ 0, 
      /* communicator = */ MPI_COMM_WORLD, 
      /* status       = */ MPI_STATUS_IGNORE);
    printf("Process 1 received number %d from process 0\n", number);
  }
  MPI_Finalize();
}
```
---
> Pateikite ir paaiškinkite deadlock susidarymo pavyzdį: 2 procesai apsikeičia pranešimais. Kaip jo išvengti?
```cpp
 if (rank == 0) {
      MPI_Send(..., 1, tag, MPI_COMM_WORLD);
      MPI_Recv(..., 1, tag, MPI_COMM_WORLD, &status);
 } else if (rank == 1) {
      MPI_Send(..., 0, tag, MPI_COMM_WORLD);
      MPI_Recv(..., 0, tag, MPI_COMM_WORLD, &status);
 }
```
Consider the following and assume that the MPI_Send does not complete until the corresponding MPI_Recv is posted and visa versa. The MPI_Send commands will never be completed and the program will deadlock.

---
> Pateikite MPI_Bcast() funkcijos realizaciją “point-to-point” duomenų siuntimo funkcijų pagalba.
```cpp
#include <stdio.h>
#include <stdlib.h>
#include <mpi.h>

void my_bcast(void* data, int count, MPI_Datatype datatype, int root,
              MPI_Comm communicator) {
  int world_rank;
  MPI_Comm_rank(communicator, &world_rank);
  int world_size;
  MPI_Comm_size(communicator, &world_size);

  if (world_rank == root) {
    // If we are the root process, send our data to everyone
    int i;
    for (i = 0; i < world_size; i++) {
      if (i != world_rank) {
        MPI_Send(data, count, datatype, i, 0, communicator);
      }
    }
  } else {
    // If we are a receiver process, receive the data from the root
    MPI_Recv(data, count, datatype, root, 0, communicator, MPI_STATUS_IGNORE);
  }
}

int main(int argc, char** argv) {
  MPI_Init(NULL, NULL);

  int world_rank;
  MPI_Comm_rank(MPI_COMM_WORLD, &world_rank);

  int data;
  if (world_rank == 0) {
    data = 100;
    printf("Process 0 broadcasting data %d\n", data);
    my_bcast(&data, 1, MPI_INT, 0, MPI_COMM_WORLD);
  } else {
    my_bcast(&data, 1, MPI_INT, 0, MPI_COMM_WORLD);
    printf("Process %d received data %d from root process\n", world_rank, data);
  }

  MPI_Finalize();
}
```
---
> Pateikite MPI_Gather() funkcijos realizaciją “point-to-point” duomenų siuntimo funkcijų pagalba.
```cpp
#include <stdio.h>
#include <stdlib.h>
#include <mpi.h>

int main(int argc, char** argv) {
	MPI_Init(NULL, NULL);

	int world_rank;
	int root = 0;
	MPI_Comm_rank(MPI_COMM_WORLD, &world_rank);
	printf("Process %d sending data %d\n", world_rank, world_rank);
	MPI_Send(&world_rank, 1, MPI_INT, root, 0, MPI_COMM_WORLD);

	if (world_rank == 0) {
		int world_size;
		MPI_Comm_size(MPI_COMM_WORLD, &world_size);
		
		for (int i = 0; i < world_size; i++) {
			int result;
			MPI_Recv(&result, 1, MPI_INT, i, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
			printf("Process 0 received data %d from %d process\n", result, i);
		}
	}

	MPI_Finalize();
}
```
---
> Pateikite MPI_Reduce() funkcijos realizaciją “point-to-point” duomenų siuntimo funkcijų pagalba.
```cpp
#include <stdio.h>
#include <stdlib.h>
#include <mpi.h>

int main(int argc, char** argv) {
	MPI_Init(NULL, NULL);

	int world_rank;
	int root = 0;
	MPI_Comm_rank(MPI_COMM_WORLD, &world_rank);
	printf("Process %d sending data %d\n", world_rank, world_rank);
	MPI_Send(&world_rank, 1, MPI_INT, root, 0, MPI_COMM_WORLD);

	if (world_rank == 0) {
		int world_size;
		MPI_Comm_size(MPI_COMM_WORLD, &world_size);
		
		int sum = 0;
		for (int i = 0; i < world_size; i++) {
			int result;
			MPI_Recv(&result, 1, MPI_INT, i, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
			printf("Process 0 received data %d from %d process\n", result, i);
			sum += result;
		}
		
		printf("final result is %d", sum);
	}

	MPI_Finalize();
}
```
---
> Pateikite algoritmą matricos ir vektoriaus sandaugos apskaičiavimui su MPI ir paaiškinkite jo vykdymą.

```cpp
#include <iostream>
#include <cstdlib>
#include "mpi.h"

using namespace std;

void multiply(int N) {
    int rank, num_of_processes;
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &num_of_processes);

    MPI_Status Stat;

    // To distribute marix rows between processes
    int aver_num_rows = N / num_of_processes;
    int extra = N % num_of_processes; // liekana, jei nesidalina po lygiai
    // compute number of rows for each process
    int num_rows = (rank < extra) ? aver_num_rows + 1 : aver_num_rows;
    double **A; // matrix

    if (rank == 0) { // Master process creates and disributes matrix A between processes
        A = new double *[N]; // allocate N rows

        for (int i = 0; i < N; i++)  // allocate N elements for i-th row
            A[i] = new double[N];

        for (int i = 0; i < N; i++) // initialize matix - assign values to elements
            for (int j = 0; j < N; j++)
                A[i][j] = rand() % 10;


        int offset = num_rows;

        for (int dest = 1; dest < num_of_processes; dest++) {
            int dest_num_rows = (dest < extra) ? aver_num_rows + 1 : aver_num_rows;
            for (int i = 0; i < dest_num_rows; i++) {
                MPI_Send(&A[offset][0], N, MPI_DOUBLE, dest, 100, MPI_COMM_WORLD);
                offset++;
            }
        }
    } else // other processes receive corresponding rows of matrix A
    {
        A = new double *[num_rows]; // allocate num_rows rows

        for (int i = 0; i < num_rows; i++)  // allocate N elements for i-th row
            A[i] = new double[N];

        for (int i = 0; i < num_rows; i++)
            MPI_Recv(&A[i][0], N, MPI_DOUBLE, 0, 100, MPI_COMM_WORLD, &Stat);
    }

    double *y; // result vector
    if (rank == 0)
        y = new double[N]; // allocate vector with N elements
    else
        y = new double[num_rows]; // allocate vector with num_rows elements

    // 1-st algorithm: multiplication by rows
	for (int i = 0; i < num_rows; i++)
		y[i] = 0;

	for (int i = 0; i < num_rows; i++)
		for (int j = 0; j < N; j++)
			y[i] += A[i][j];

    if (rank != 0) // All other processes send its part of result vector y[]
        MPI_Send(y, num_rows, MPI_DOUBLE, 0, 200, MPI_COMM_WORLD);
    else { // process 0 receives all parts of result vector y[]
        int offset = num_rows;

        for (int dest = 1; dest < num_of_processes; dest++) {
            int dest_num_rows = (dest < extra) ? aver_num_rows + 1 : aver_num_rows;
            MPI_Recv(&y[offset], dest_num_rows, MPI_DOUBLE, dest, 200, MPI_COMM_WORLD, &Stat);
            offset = offset + dest_num_rows;
        }
    }

    // free memory
    if (rank == 0)
        for (int i = 0; i < N; i++)
            delete[] A[i]; // deallocate N elements for i-th row
    else
        for (int i = 0; i < num_rows; i++)
            delete[] A[i]; // deallocate N elements for i-th row

    delete[] A;

    delete[] y; // deallocate vector y with N elements
}

int main(int argc, char *argv[]) {
    MPI_Init(&argc, &argv);

    multiply(1000);
	
    MPI_Finalize();
    return 0;
}

```