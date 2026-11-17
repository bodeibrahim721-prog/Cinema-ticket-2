# Cinema-ticket-2
Cinema tickets java project 
import java.util.concurrent.Semaphore;

class Customer extends Thread {

    int id;

    Customer(int id) {
        this.id = id;
    }

    public void run() {

        for (int i = 0; i < 8; i++) {

            boolean entered = false;

            try {

                Main.stations.acquire();
                entered = true;

                Main.mutex.acquire();

                Main.busy++;

                if (Main.busy > Main.maxBusy) {
                    Main.maxBusy = Main.busy;
                }

                Main.mutex.release();

                Main.mutex.acquire();

                if (Main.stock > 0) {
                    Main.stock--;
                    Main.served++;
                } else {
                    Main.rejected++;
                }

                Main.mutex.release();

                Thread.sleep(10);

            } catch (InterruptedException e) {
                return;

            } finally {

                if (entered) {
                    try {
                        Main.mutex.acquire();

                        Main.busy--;

                        Main.mutex.release();

                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                    }

                    Main.stations.release();
                }
            }
        }
    }
}

public class Main {

    static int stock = 40;
    static int served = 0;
    static int rejected = 0;

    static int busy = 0;
    static int maxBusy = 0;

    static Semaphore mutex = new Semaphore(1);
    static Semaphore stations = new Semaphore(2);

    public static void main(String[] args) {

        Customer[] customers = new Customer[6];

        for (int i = 0; i < 6; i++) {
            customers[i] = new Customer(i + 1);
            customers[i].start();
        }

        for (int i = 0; i < 6; i++) {
            try {
                customers[i].join();
            } catch (InterruptedException e) {
                return;
            }
        }

        System.out.println("Tickets sold: " + served);
        System.out.println("Customers turned away: " + rejected);
        System.out.println("Seats left: " + stock);
        System.out.println("Most windows busy at the same time: " + maxBusy);
    }
}
