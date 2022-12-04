
public class Main {

    public static void main(String[] args) {
        // TODO code application logic here

        int n;
        Random rand = new Random();
        float CGPA;

        Scanner ler = new Scanner(System.in);
        System.out.print("|----------------Welcome to FutureBuilder---------------------|\n");
        System.out.print(" 1.Enter the number of students 1, 10, 100, 1000 or 10000: \n");
        n = ler.nextInt();


        //Create two students array
        Student[] ArrayOfStudents1 = new Student[n];
        Student[] ArrayOfStudents2 = new Student[n];


        //Fill the arrays with random CGPA
        for( int i = 0; i < n; i++)
        {
            CGPA = rand.nextFloat() * 10.0f;

            ArrayOfStudents1[i] = new Student(CGPA);

            ArrayOfStudents2[i] = new Student(CGPA);
        }

        //Print unsorted array 
        print(ArrayOfStudents1);

        long t = System.currentTimeMillis(); //take start time
        noParalleOddEvenSort(ArrayOfStudents1, n); //sort array
        t = System.currentTimeMillis() - t; //take end time

        System.out.println("\nSorted Students CGPA:");

        print(ArrayOfStudents1); //Print sorted array 

        System.out.println("Time spent for No Paralle Odd Even Transposition Sort: " + t + "ms");

        long t1 = System.currentTimeMillis();
        paralleOddEvenSort(ArrayOfStudents2, n);
        t1 = System.currentTimeMillis() - t1;

        System.out.println("\nSorted Students CGPA:");

        print(ArrayOfStudents2);

        System.out.println("Time spent for Paralle Odd Even Transposition Sort: " + t + "ms");
    }

    static void print(Student arr[])
    {
        System.out.print("CGPA = [ ");
        for( int i = 0; i < arr.length ; i++)
            System.out.printf("\n%.3f ",arr[i].getCGPA() );
        System.out.print("]\n");
    }


    /*

     */
    static void noParalleOddEvenSort(Student arr[], int n)
    {
        boolean isSorted = false;

        while (!isSorted) {
            isSorted = true;
            Student temp;

            //Perform the bubble sort in odd phase
            for (int i = 1; i <= n - 2; i = i + 2) {
                if (arr[i].CGPA > arr[i + 1].CGPA) {
                    temp = arr[i];
                    arr[i] = arr[i + 1];
                    arr[i + 1] = temp;
                    isSorted = false;
                }
            }


            //Perform the bubble sort in even phase
            for (int i = 0; i <= n - 2; i = i + 2) {
                if (arr[i].CGPA > arr[i + 1].CGPA) {
                    temp = arr[i];
                    arr[i] = arr[i + 1];
                    arr[i + 1] = temp;
                    isSorted = false;
                }
            }
        }

        return;
    }

    static void paralleOddEvenSort(Student arr[], int n)
    {
        if(n == 1) return;

        int threadNum = arr.length/2;

        CyclicBarrier barr = new CyclicBarrier(threadNum);

        //Creating Threads
        Thread[] threads = new Thread[threadNum];
        for (int i = 0; i < threadNum; i++) {
            threads[i] = new Thread(new CompareSwapThread(arr, 2*i + 1, barr));
            threads[i].start();
        }

        //Waiting  for all the threads to finish
        for (int i = 0; i < threadNum; i++) {
            try {
                threads[i].join();
            } catch (InterruptedException e) {e.printStackTrace();}
        }
    }
}

class CompareSwapThread implements Runnable {
private Student[] arr;
private int index;
private CyclicBarrier barr;

    //Compare and exchange the consecutive elements of the array
    public CompareSwapThread(Student[] arr, int index, CyclicBarrier barr) {
        this.arr = arr;
        this.index = index;
        this.barr = barr;
    }

    // Each thread compares
    // two consecutive elements of the array
    @Override
    public void run() {
        for (int i = 0; i < arr.length; i++) {
            if (arr[index - 1].CGPA > arr[index].CGPA) {
                Student temp = arr[index - 1];
                arr[index - 1] = arr[index];
                arr[index] = temp;
            }
            try {
                barr.await();
            } catch (InterruptedException | BrokenBarrierException e) {e.printStackTrace();}
            if (index + 1 < arr.length && arr[index].CGPA > arr[index + 1].CGPA) {
                Student temp = arr[index];
                arr[index] = arr[index + 1];
                arr[index + 1] = temp;
            }
            try {
                barr.await();
            } catch (InterruptedException | BrokenBarrierException e) {e.printStackTrace();}
        }
    }
}

class Student
{
float CGPA;

    public Student(float CGPA) {
        this.CGPA = CGPA;
    }
    public float getCGPA() {
        return this.CGPA;
    }
    public void setCGPA(float CGPA) {
        this.CGPA = CGPA;
    }
}