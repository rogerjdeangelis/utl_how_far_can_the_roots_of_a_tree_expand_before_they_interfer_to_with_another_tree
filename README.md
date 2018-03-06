# utl_how_far_can_the_roots_of_a_tree_expand_before_they_interfer_to_with_another_tree
How far can the roots of a tree expand before they interfer to with another tree. This has many applications. 
    How far can the roots of a tree expand before they interfer to with another tree.

    SAS/WPS/R: How far can the roots of a tree expand before they interfer to with another tree.

    my high res input plot of data
    https://tinyurl.com/yb93aptx
    https://github.com/rogerjdeangelis/utl_how_far_can_the_roots_of_a_tree_expand_before_they_interfer_to_with_another_tree/blob/master/utl_how_far_can_the_roots_of_a_tree_expand_before_they_interfer_to_with_another_tree.pdf

    Sources
    https://goo.gl/rULaF8
    http://stackoverflow.com/questions/42787711/how-can-i-get-the-area-of-each-voronoi-polygon-in-r

    https://goo.gl/Mc1urr
    http://flowingdata.com/2016/04/12/voronoi-diagram-and-delaunay-triangulation-in-r/

    https://goo.gl/05hEhl
    http://letstalkdata.com/2014/05/creating-voronoi-diagrams-with-ggplot/

    What is the area of influence of tree roots?

    The plot below is probably distorted depending on
    the font. But the distance from Tree to
    the T is bisected by the boundary.


    HAVE  Loaction of trees
    =======================

                                         NO LOSS OF GENERALITY
             LATITUDE_      LOGITUDE_    FOR EASY CHECKING
    Obs        DEGREES        DEGREES    LAT    LONG

     1       30.000030      90.000090     30      90
     2       30.000030      90.000110     30     110
     3       30.000040      90.000100     40     100
     4       30.000050      90.000090     50      90
     5       30.000050      90.000110     50     110

    LONGitude
    micro degrees
    120 +------------------+
        |         |        |
        |         |        |
        |         |        |
        |         |        |
    110 |   Tree  /\   Tree|
        |        /  \      |
        |       /    \     |
        |      /      \    |
        |     /        \   |
    100 +--- /  Tree   /---+
        |    \        /    |
        |     \      /     |
        |      \    /      |
        |       \  /       |
     90 |   Tree \/    Tree|
        |        |         |
        +        |         +
        -+---+------------+
            30  40    50

              LATitude
            micro degrees

    WANT
    ====

    AREA_INFLUENCE=200  micro degrees squared  (outside triangle)

    Area of influence for Tree located at longitude=100  latitude=40
    is AREA_INFLUENCE=200  micro degrees squared

    DETAILS
    =======

    Let's calculate the area of influence for central tree
    We have two triangles.

    Area=Two triangles= 2 * ( 1/2 * base*height) =base,height

    Base is 20 degrees  (50-30) =20
    Height 10 degrees  (100-90) =10

    Area=20*10=200 (square micro degrees)

    RECTANGLES ON THE SPHERE

    On the earths surface the area is a spherical rectangle.

    http://mathforum.org/library/drmath/view/63767.html

    Where R is the radius of the earth. I am ignoring this.
    I think this formula uses degrees not radians?

    A = 2*pi*R^2 |sin(lat1)-sin(lat2)| |lon1-lon2|/360
        = (pi/180)R^2 |sin(lat1)-sin(lat2)| |lon1-lon2|

    I think the result is in units of the Radius  (usually square kilometers)

    Sum of the angles of a sherical triangle > 360 degrees.

    WORKING CODE
    ============
       WPS/R

          voronoi <- deldir(df$LONG, df$LAT);

    FULL SOLUTION
    ============

    * create data in micro degrees;
    options validvarname=upcase;
    libname sd1 "d:/sd1";
    data sd1.have(rename=(y=long x=lat));;

     retain latitude_degrees lat logitude_degrees long;
     format latitude_origin_degrees logitude_origin_degrees 11.6;
     array  ys[5] _temporary_ (90,110,100,90,110);
     array  xs[5] _temporary_ (30,30,40,50,50);
     do i=1 to dim(xs);
       x=xs[i];
       y=ys[i];
       latitude_degrees = 30 + x/1e6;
       logitude_degrees = 90 + y/1e6;
       output;
     end;
     keep  x y latitude_degrees logitude_degrees;
    run;quit;

    proc print data=sd1.have;
    format latitude_degrees logitude_degrees 11.6;
    run;quit;


    options ls=171;
    %utl_submit_wps64('
    options set=R_HOME "C:/Program Files/R/R-3.3.2";
    libname wrk "%sysfunc(pathname(work))";
    proc r;
    submit;
    source("C:/Program Files/R/R-3.3.2/etc/Rprofile.site", echo=T);
    library(haven);
    library(deldir);
    library(ggplot2);
    df<-read_sas("d:/sd1/have.sas7bdat");
    df;
    voronoi <- deldir(df$LONG, df$LAT);
    area<-voronoi$summary;
    area;
    endsubmit;
    run;quit;
    ');

    * If you want the plot;
    %utl_submit_wps64('
    options set=R_HOME "C:/Program Files/R/R-3.3.2";
    libname wrk "%sysfunc(pathname(work))";
    proc r;
    submit;
    source("C:/Program Files/R/R-3.3.2/etc/Rprofile.site", echo=T);
    library(haven);
    library(deldir);
    library(ggplot2);
    df<-read_sas("d:/sd1/have.sas7bdat");
    df;
    voronoi <- deldir(df$LONG, df$LAT);
    area<-voronoi$summary;
    pdf("d:/pdf/utl_how_far_can_the_roots_of_a_tree_expand_before_they_interfer_to_with_another_tree.pdf");
    ggplot(data=df, aes(x=LONG,y=LAT)) +
      geom_segment(
        aes(x = x1, y = y1, xend = x2, yend = y2),
        size = 2,
        data = voronoi$dirsgs,
        linetype = 1,
        color= "#FFB958") +
      geom_point(
        fill=rgb(70,130,180,255,maxColorValue=255),
        pch=21,
        size = 4,
        color="#333333");
    endsubmit;
    import r=area data=wrk.area;
    run;quit;
    ');

    FROM R
    Up to 40 obs from area total obs=5

    Obs     X      Y    N_TRI    DEL_AREA    DEL_WTS    N_TSIDE    NBPT    DIR_AREA    DIR_WTS

     1      90    30      2        66.667    0.16667       3         2         94      0.16319
     2     110    30      2        66.667    0.16667       3         2         94      0.16319
     3     100    40      4       133.333    0.33333       4         0        200      0.34722
     4      90    50      2        66.667    0.16667       3         2         94      0.16319
     5     110    50      2        66.667    0.16667       3         2         94      0.16319


    Data want;
      set area(where=(x=100 and y=40) keep=dir_area x y);
      Area_influence=dir_area;
      put "Area of influence for Tree located at longitude=" x " latitude=" y " is " area_influence= " micro degrees squared";
    run;quit;

    Area of influence for Tree located at  X=100 Y=40  is AREA_INFLUENCE=200  micro degrees squared


    LOG

    > source("C:/Program Files/R/R-3.3.2/etc/Rprofile.site", echo=T);
    > .libPaths(c(.libPaths(), "d:/3.3.2", "d:/3.3.2_usr"))
    > options(help_type = "html")
    > library(haven);
    > library(deldir);
    deldir 0.1-12
    > library(ggplot2);
    > df<-read_sas("d:/sd1/have.sas7bdat");
    > df;
    > voronoi <- deldir(df$LONG, df$LAT);
         PLEASE NOTE:  The components "delsgs" and "summary" of the
     object returned by deldir() are now DATA FRAMES rather than
     matrices (as they were prior to release 0.0-18).
     See help("deldir").


    The WPS System

    # A tibble: 5 × 4
      LATITUDE_DEGREES LOGITUDE_DEGREES   LAT  LONG
                 <dbl>            <dbl> <dbl> <dbl>
    1         30.00003         90.00009    30    90
    2         30.00003         90.00011    30   110
    3         30.00004         90.00010    40   100
    4         30.00005         90.00009    50    90
    5         30.00005         90.00011    50   110


    NOTE: Processing of R statements complete

    17        import r=area data=wrk.area;
    NOTE: Creating data set 'WRK.area' from R data frame 'area'
    NOTE: Column names modified during import of 'area'
    NOTE: Data set "WRK.area" has 5 observation(s) and 9 variable(s)

    18        run;
    NOTE: Procedure r step took :
          real time : 1.210
          cpu time  : 0.015



