# The Analysis Codes

Author: ***MITRADIP DAS***, NISER, Bhubaneswar, India.

**Table of Contents:**

* [Compiler Information](#compiler)
* [Documentation](#doc)
    * [libraries.h](#lib)
    * [analysis.cpp](#anls)
    * [rdf_1.cpp](#rdf)
    * [msd_1.cpp](#msd)
    * [intr_1.cpp](#intr)
    * [tcf_1.cpp](#tcf)
* [Designing your own library](#design)

## <a name="compiler"></a> Compiler Information:

The codes were compiled and tested using **clang++-5.0** using **c++14 standard** in **Linux 4.8** (Linux Mint 18.2). The files _analysis.cpp_ and _cpp_include/libraries.h_ **should not** be compiled as stand alone files and are meant to be used as libraries. For running the code, kindly compile the files with _main()_ functions only (like _rdf_1.cpp_) as per the requirements. You are also invited to design your own libraries, the instructions for which are given later in this file.

## <a name="doc"></a>Documentation: 

### <a name="lib"></a>File: cpp_include/libraries.h

This file contains different libraries and generic functions used in the code. It references to different standard c++ libraries like _iostream_, _fstream_ etc. The generic functions contained are:

*   **std::string trim_left(const std::string& str):** This function left trims a string _str_.
*   **std::string trim_right(const std::string& str)**: This function right trims a string _str._
*   **std::string trim(const std::string& str):** This function trims a string _str_ using _trim_left_ and _trim_right._
*   **long int line\_skip(string file\_name,long int start_pos,long int lines):** This functions returns the position of the get pointer in file _file_name_ after skipping _lines_ lines starting from position of get pointer _start_pos_.

### File: analysis.cpp <a name="anls"></a>

This file contains the codes for analysis of the system. It has different classes for different sections of the work. This file implements _cpp_include/libraries.h_.

*   **class atom:** Each object of this class represents an atom and it contains properties of atom like ID, name, Residue (molecule) ID and coordinates.

    *   **void show():** This is used to display the atom and the above properties.

*   **class frame:** This frame represents one frame from a run of MD simulation. It contains information of multiple atoms (implemented as a vector) and the corresponding box length of the frame.

    *   **long int read\_file(string file\_name,long int start_line):** This function reads the atoms from a PDB file _file_name_ generated using VMD (implementing PBC) with get pointer from _start_line_ till "END" is encountered. It returns the current position of the get pointer in the file (useful while implementing multiple frames).
    *   **void clear_all()**: Clear all the information present in a frame.
    *   **void set_box(double len):** Sets the box length to _len._
    *   **void show_frame():** Display all the data of the whole frame.
    *   **double calc\_dist\_pbc(long int id1,long int id2):** This function returns the distance between two atoms with _id1_ and _id2_ implementing PBC.
    *   **void get\_idx\_of(string src,std::vector\<int\> &v):** This functions stores all the index of atoms with name _src_ in referenced vector _v_.
    *   **void pair\_rdf\_fn(double start, double stop, double interval, const string pairs\[\], std::vector <array <double,2>> &rdf):** This function calculates the RDF of the current frame between atom types given in _pairs\[\]_. The variables _start, stop_ and _interval_ defines the start, end and mesh size of the RDF, in referenced vector _rdf._ The RDF implements PBC conditions and hence non-zero results exists only up to half of box length.
    *   **bool interact\_2(long id1,long id2,double d12l,double d12h):** This function checks if two atoms, given by indices _id1_ and _id2_ interact via a distance between _d12l_ and _d12h_.
    *   **bool interact\_3(long id1,long id2,long id3,double d12l,double d12h,double d23l,double d23h,double a123l,double a123h):** This function checks if three atoms, with id's _id1_, _id2_ and _id3_ are interacting such that the distance between 1 & 2 lies between _d12l_ and _d12h_, distance between 2 & 3 lies between _d23l_ and _d23h_ and angle 123 (in degrees) is between _a123l_ and _a123h_.
    *   **long cnt\_intr\_2(const string atm1,const string atm2,double d12l,double d12h):** This function counts the number of atoms of types _atm1_ and _atm2_ interacting via a distance between _d12l_ and _d12h_.
    *   **long cnt\_intr\_3(const string atm1,const string atm2,const string atm3,double d12l,double d12h,double d23l,double d23h,double a123l,double a123h):** This function counts the number interacting atoms with atom types _atm1_, _atm2_ and _atm3_ such that the distance between 1 & 2 lies between _d12l_ and _d12h_, distance between 2 & 3 lies between _d23l_ and _d23h_ and angle 123 (in degrees) is between _a123l_ and _a123h_.
    *   **void list_intr_2(std::vector\<int> id1,std::vector\<int> id2,double d12l,double d12h,std::vector\<int> &idx1,std::vector\<int> &idx2):** Checks the lists _id1_ and _id2_ with atomic ID's for interactions (such that the distance is between _d12l_ and _d12h_) and stores the same in _idx1_ and _idx2_.
 
*   **class trajectory:** This class represents the trajectory of the atoms and implements a vector of frames in order to achieve this target.

    *   **long int read\_frames(string file\_name, long int st_pos, long int nframes, double boxl):** This function is used to read _nframes_ frames from a PDB file _file_name_ generated by VMD starting from file get pointer _st_pos_. This function also fixes the box length of each frame to _boxl_ and it return the current position of get pointer after reading the frames.
    *   **void pair\_rdf\_fn(double start, double stop, double interval, const string pairs\[\], std::vector <array<double,2>> &rdf):** This function gives the time average of RDF implementing _pair\_rdf\_fn_ from class frame.
    *   **double idx\_msd\_fn(int dt,const int idx):** This function calculates and returns the MSD of atom present in index _idx_ for a frame (or time) interval _dt_.
    *   **double type\_msd\_fn(int dt,const string src):** This function calculates and returns the MSD of an atom type _src_ over the whole ensemble for a frame (or time) interval _dt_. This implements the _idx\_msd\_fn_.
    *   **void type\_msd\_fn\_all\_dt(const string src,std::vector<array<double,2>> &v):** This function calculates the overall MSD for all possible frame intervals, 1 to (no of frames-1), for a given atom type _src_ and stores it in the reference vector _v_.
    *   **void intr\_cnt\_3(const string atm1,const string atm2,const string atm3,double d12l,double d12h,double d23l,double d23h,double a123l,double a123h,std::vector<array<long,2>> &v):** This function counts the number interacting atoms with atom types _atm1_, _atm2_ and _atm3_ in each time step such that the distance between 1 & 2 lies between _d12l_ and _d12h_, distance between 2 & 3 lies between _d23l_ and _d23h_ and angle 123 (in degrees) is between _a123l_ and _a123h_, and stores the data in vector _v_.
    *   **void intr\_cnt\_2(const string atm1,const string atm2,double d12l,double d12h,std::vector<array<long,2>> &v):** This function counts the number of atoms of types _atm1_ and _atm2_ in each time step interacting via a distance between _d12l_ and _d12h_.
    *   **void tcf\_2\_all\_dt\_im(const string atm1,const string atm2,double d12l,double d12h,std::vector<array<double,2>> &v):** Gives the time correlation function (survival proablity) between atom types _atm1_ and _atm2_ interacting via distance between _d12l_ and _d12h_, with intermediate approximation, and stores it in vector _v_ for all time steps.
    *   **void tcf\_2\_all\_dt(const string atm1,const string atm2,double d12l,double d12h,std::vector<array<double,2>> &v) :** Gives the time correlation function (survival proablity) between atom types _atm1_ and _atm2_ interacting via distance between _d12l_ and _d12h_, without intermediate approximation, and stores it in vector _v_ for all time steps.

### <a name="rdf"></a> File: rdf_1.cpp 

This file contains the _main()_ function demonstrating a sample RDF run.

### <a name="msd"></a>File: msd_1.cpp

This file contains the _main()_ function demonstrating a sample MSD run.

### <a name="intr"></a>File: intr_1.cpp

This file contains the _main()_ function demonstrating a sample interactional statistics study.

### <a name="tcf"></a>File: tcf_1.cpp

This file contains the _main()_ function demonstrating a sample interactional dynamics (time correlation function) study without intermediate approximation.

## <a name="design"></a>Designing your own library 

You are always invited to design your own libraries on this code and contact the author about the same, if you want it to be implemented. Please download _analysis.cpp_ and _cpp\_include/libraries.h_ and generate your own C++ code for the analysis required. You may consider looking at the sample codes for reference. Please note that _libraries.h_ has to be inside _cpp\_include_ directory.
