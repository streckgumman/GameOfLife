# GameOfLife
package exercises;

import javafx.animation.AnimationTimer;
import javafx.application.Application;
import javafx.scene.Group;
import javafx.scene.Scene;
import javafx.scene.canvas.Canvas;
import javafx.scene.canvas.GraphicsContext;
import javafx.scene.paint.Color;
import javafx.scene.shape.Circle;
import javafx.stage.Stage;

import java.util.Arrays;
import java.util.Random;

import static java.lang.Math.round;
import static java.lang.Math.sqrt;
import static java.lang.System.*;

/*
 * Program for Conway's game of life.
 * See https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life
 *
 * This is a graphical program using JavaFX to draw on the screen.
 * There's a bit of "drawing" code to make this happen (far below).
 * You don't need to implement (or understand) any of it.
 * NOTE: To run tests must uncomment in init() method, see comment
 *
 * Use smallest step development + functional decomposition!
 *
 * See:
 * - UseEnum
 * - BasicJavaFX (don't need to understand, just if you're curious)
 */
public class Ex5GameOfLife extends Application {

    final Random rand = new Random();

    // Enum type for state of Cells
    enum Cell {
        DEAD, ALIVE
    }
    /*

                                 init()
                                    |
                   ...................................
                   |                |               |
          Cell[]getCells()      shuffle        world = toMatrix

    */

    // This is the *only* accepted modifiable instance variable in program except ...
    Cell[][] world;

    public void init() {
        //test();        // <--------------- Uncomment to test!
        int nLocations = 9000;
        double distribution = 0.4;   // % of locations holding a Cell
        // TODO create world
        Cell[] cs = getCells(nLocations, distribution);
        shuffle(cs);
        world = toMatrix(cs);

        //getCells(nLocations, distribution);
        /// Testing...2};
        /*world = new Cell[2][3];
        world[0][0] = Cell.DEAD;
        world[0][1] = Cell.ALIVE;
        world[0][2] = Cell.ALIVE;
        world[1][0] = Cell.DEAD;
        world[1][1] = Cell.ALIVE;
        world[1][2] = Cell.DEAD;
        */



    }

    Cell[][] toMatrix(Cell[] arr){
        int size = (int) round(sqrt(arr.length));
        Cell[][] matrix = new Cell[size][size];
        for (int i = 0; i < arr.length; i++){
            matrix[i/size][i%size] = arr[i];

        }

        return matrix;
    }


    Cell[] getCells(int nCells, double percentAlive){
        Cell[] cells = new Cell[nCells];
        int nAlive = (int) Math.round(percentAlive * nCells);
        int i = 0;
        while (nAlive > 0){
            cells[i] = Cell.ALIVE;
            nAlive--;
            i++;
        }
        while(i < nCells){
            cells[i] = Cell.DEAD;
            i++;
        }
        return cells;
    }

    void shuffle(Cell[] arr){
        for (int i = arr.length; i > 1; i--){
            int j = rand.nextInt(i);
            Cell temp = arr[j];
            arr[j] = arr[i-1];
            arr[i-1] = temp;
        }
    }




    long timeLastUpdate;   // ... this, used for speed of animation.

    // Implement this method (using functional decomposition)
    // Every involved method should be tested, see below, method test()
    // This method is automatically called by a JavaFX timer (don't need to call it)
    /*
    BÖRJA NERIRÅN

                                    update()
                                        |
                                     ..............
                                               |
                                        world = nextState           //gör ny värld
                                                   |
                                  ...................................................
                                  |                              |                  |
             forAll nAlive = getLivingNeighbours         switch(nAlive)         return world
                                    |
                              isValidLocation()


    */


    void update(long now) {
        if (now - timeLastUpdate > 500_000_000) {  // Has time passed?
            // TODO update world
            world = nextState(world);
            timeLastUpdate = now;
        }
    }


    // -------- Write methods below this --------------
    Cell[][] nextState(Cell[][] world) {
        int size = world.length;
        Cell[][] newWorld = new Cell[size][size];
        for (int row = 0; row < size; row++){
            for (int column = 0; column < size; column++){
                int nAlive = getLivingNeighbours(world, row, column);
                if (nAlive > 3){
                    newWorld[row][column] = Cell.DEAD;
                } else if (nAlive == 3){
                    newWorld[row][column] = Cell.ALIVE;
                } else if (nAlive < 2){
                    newWorld[row][column] = Cell.DEAD;
                } else {
                    newWorld[row][column] = newWorld[row][column];
                }
            }
        }

        return newWorld;
    }


    boolean isValidLocation(int size, int row, int col){
        return 0 <= row && row < size && 0<= col && col < size;
    }

    int getLivingNeighbours(Cell[][] world, int row, int col){          //hårig kod
        int count = 0;
        for (int r = row - 1; r <= row + 1; r++){
            for (int c = col - 1; c <= col + 1; c++ ){
                if (isValidLocation(world.length, r, c) && !(r == row && c == col)) {
                    if (world[r][c] == Cell.ALIVE){
                        count++;
                    }
                }
            }
        }
        return count;
    }



    // ---------- Testing -----------------
    // Here you run your tests i.e. call your logic methods
    // to see that they really work
    void test() {
        // Hard coded test world
        Cell[][] testWorld = {
                {Cell.ALIVE, Cell.ALIVE, Cell.DEAD},
                {Cell.ALIVE, Cell.DEAD, Cell.DEAD},
                {Cell.DEAD, Cell.DEAD, Cell.ALIVE},

        };
        int size = testWorld.length;

        //out.println(getLivingNeighbours(testWorld, 0, 0));
        //out.println(getLivingNeighbours(testWorld, 1, 1));
        //out.println(isValidLocation(size, 0, 0));
        //out.println(!isValidLocation(size, -1, 0));
        //out.println(!isValidLocation(size, 0, size));
        //out.println(Arrays.toString(getCells(10, 0.5)));
        //Cell[] cs = getCells(20, 0.25);
        //shuffle(cs);
        //out.println(Arrays.toString(cs));


        // TODO


        exit(0);
    }

    // -------- Below is JavaFX stuff, nothing to do --------------

    void render() {
        gc.clearRect(0, 0, width, height);
        int size = world.length;
        for (int row = 0; row < size; row++) {
            for (int col = 0; col < size; col++) {
                int x = 10 * col + 50;
                int y = 10 * row + 50;
                renderCell(x, y, world[row][col]);
            }
        }
    }

    void renderCell(int x, int y, Cell cell) {
        if (cell == Cell.ALIVE) {
            gc.setFill(Color.RED);
        } else {
            gc.setFill(Color.WHITE);
        }
        gc.fillOval(x, y, 10, 10);
    }

    final int width = 400;   // Size of window
    final int height = 400;

    GraphicsContext gc;

    // Must have public before more later.
    @Override
    public void start(Stage primaryStage) throws Exception {

        // JavaFX stuff
        Group root = new Group();
        Canvas canvas = new Canvas(width, height);
        root.getChildren().addAll(canvas);
        gc = canvas.getGraphicsContext2D();

        // Create a timer
        AnimationTimer timer = new AnimationTimer() {
            // This method called by FX at a certain rate, parameter is the current time
            public void handle(long now) {
                update(now);
                render();

            }
        };
        // Create a scene and connect to the stage
        Scene scene = new Scene(root);
        primaryStage.setScene(scene);
        primaryStage.setTitle("Game of Life");
        primaryStage.show();
        timer.start();  // Start simulation
    }

    public static void main(String[] args) {
        launch(args);   // Launch JavaFX
    }
}
