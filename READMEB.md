# Gabella notes.
On Linux compile this with GNU gfortran
```
gfortran AR-CHAINWH.f -o AR-CHAINWH
```
The shell script runarchainWH does a check for an input file and will also
output some timing file, time.out by default.  Also check the hardcoded
line for the AR-CHAINWH executable, typically with GWToolsWormholes it will
be ./ARCHAIN .


# DATA EXAMPLE:
  0  5   .1d0    20000.   0.e-3    0.0   1. 1.e-99 0.e0 1.e4  X  1 0  0.0 .70 0.3  1.e-13 0  10000 /IWR,N,DELTAT,TMAX ,step,soft,cmet,Clight, outfile,Ixc,Nbh ,spin, EPS, ivelocity, KSMX ! AT THE END OF THIS EXAMPLE FILE THERE ARE SOME EXPLANATIONS, please take a look!
  1500000  0.  0.  0.  0.  0.  0. 0    m x y z vx vy vz  soft (and the same for all bodies)
  50000.  39.399994  9.299988  0. -.5800018  .97399998  0.  
  10.  191.  32.5  0. -.55600004  .84599991  0. 
  10.  189. -84.700012  0.  .0033600006  .0017600002  0. 
  0  0  0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0  /STOP ; this would set N=4 even if in first line N=5 !! REMOVE THIS IF U WANT MORE BODIES
  10.  94.099976 -122.  0.  .90300007  .07999992  0. 
  10.  41.799988 -13.  0.  .04699993  0.964799976  0. 
  0  0  0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 /STOP | 

# PLEASE read these expalantions and ALSO look the comments in the code.

## FIRST DATA LINE: 

 UNITS: G=1,  I usually use Solar masses, astronomical units (AU), then the time is such that:  1 Year=2 Pi, i.e. years=time/twopi

 IWR =output index (the larger the more diagnostic output)

 N=number of bodies

 DELTAT = (approx) output interval

 TMAX =maximum time

 step = initial stepsize  (if step=0, the code attempts to estimate a reasonable one), NORMALLY =0

 soft = softening (code works best for soft=0, but softening is not prohibited), NORMALLY = 01

 cmet =three-dimesional vector that defines the used time-transformation  1 0 0 may usually be the best
         (but a small value for cmet(2) may be advisable in case of star-star near collision).

 Clight= speed of light. The code system of units is such that G=1. If AU and M_sun are =1, then CLIGHT \approx 10^4.

 outfile= name of outputfile for time and coords

 IXC= index that tels if exact output times are required, 
      0 => no exact, output when t>tnext, (fast, but sometimes output interval too long) 
      1 => exact time (not always accureate, but faster than the exact iteration)
      2 => exact iteration (time to tnext) (slower!)

 NBH= number of black holes, (which MUST be the bodies # 1,2,.., NBH)

 spin= three-dimensional vector= the (relative) spin of the BH = M1 (i.e. |psin|<1, and only the first body 
       is allowed to be a Kerr-hole, other possible BH's are non-rotating).
 
 EPS = error tolerance for the integration

 ivelocity = index telling that there are v-dependent perturbations (=1) or not (=0)

 KSMX = tells how many (maximum) integration steps the code is at most allowd to take between outputs 
        (this avoidsi almost infinity loops is step has got an unreasonably small valuei, from which it may not recover)

## ---------   END OF FIRST DATA LINE EXPLANATION ----------------

 AFTER the first line of data there is the list of masses and initial conditions
    m x y z  vx vy vz
 for all the bodies from 1 to N.


## ------------------------------------------------------------

 After finishin one simulation the code attempt to read data for the next one.
 It stops when the 1. line is zeros (actually if N<2).

# OUTPUT: (at the moment)

The coordinates goto the OUTPUT file (coords for merged bodies get huge values for moviei/picture makingi i.e. they go out of figure)  
        write(66,234)time,
     & (xwr(k),xwr(k+1),xwr(k+2),k=1,3*n_ini,3)

In addition orbital elements (Keplerian ones) are written to files 71, 72,.. 75 as explaned here:
           write(71,171)time,(ai(k),k=2,N_ini) ! a   write orbital elements (with respect to M1)
           write(72,171)time,(ei(k),k=2,N_ini) ! e
           write(73,171)time,(unci(k),k=2,N_ini) ! i
           write(74,171)time,(Omi(k),k=2,N_ini)  ! \Omega
           write(75,171)time,(ooi(k),k=2,N_ini)  ! \omega
           write(76,*)time,spin,dsp ! spin(k), k=1,3 of M1  (|spin|<1),
                                    ! dsp is error in the length of the spin vector
 

 One may change the main program easily to get other quantities computed from the coords/vels that 'ARC' returns.
 Note that the number of bodies may change if the BH swollows some of them.
  
#  LIST OF SUBROUTINES & FUNCTIONS

         subroutine ARCparams_Dot_CH_is_here ! This subroutine is not used, but the content is simply what is supposed to be in the file  ARCparams.CH.
         Program ARCcode !  This is the main program, not subroutine.
         subroutine Diagnostic output(time,xwr,cmxx)
         subroutine Write Elements(time,iopen) ! with respect to M(1)
         Subroutine MERGE_I1_I2(time)!Merge the colliding body with the BH, ('time' is here only for write(66....)
         SUBROUTINE ARC ! This is the original Alforithmic Regularization Chain code.
         subroutine Iterate2ExactTime(Y0,Nvar,deltaT,f1,d1,f2,d2,x1,x2)
         subroutine LEAPFROG(STEP,Leaps,stime) !Leapfroging steps for the Bulirsch-Stoer extrapolation
         function Wfunction() ! avluate the TTL-function for time tranformation !  not  subroutine
         subroutine omegacoef ! coefficients in \Omega=sum omec(i,j)/r_{ij}
         SUBROUTINE XCMOTION(hs) ! movement of coordinates
         subroutine PUT V 2 W ! these parameters are needed if there are v-ependent forces
         subroutine Velocity Dependent Perturbations ! This name tells what this is!
         SUBROUTINE CHECK SWITCHING CONDITIONS(MUSTSWITCH)! Is it necessary to construct a new CHAIN?
         SUBROUTINE FIND CHAIN INDICES ! Here the new indecies along the new CHAIN are obtained
         SUBROUTINE CHECK CONNECTION(IC,LMIN,LMAX,IJ,LI,SUC,LOOP) ! Loops in CHAIN are not allowed
         SUBROUTINE ARRANGE(N,Array,Indx) ! this is a sorting routine (needed in CHAIN construction)
         SUBROUTINE INITIALIZE XC AND WC  ! find CHAIN coordinates and velocities
         SUBROUTINE UPDATE X AND V ! Normal physical x & v from CHAIN XC and WC
         SUBROUTINE CHAIN TRANSFORMATION ! New CHAIN from the old one
         FUNCTION SQUARE(X,Y) ! |X-Y|^2 !  not  subroutine
         SUBROUTINE DIFSYAB(N,EPS,S,h,t,Y) ! BULIRSCH-STOER EXTRAPOLATOR (i.e. with rationals)
         subroutine SubSteps(Y0,Y,H,Leaps)! substeps for DIFSYAB
         subroutine Initialize increments 2 zero
         subroutine Take Increments 2 Y(Y)
         subroutine Put Y to XC WC (Y,Lmx)
         subroutine Take Y from XC WC (Y,Nvar)
         subroutine Obtain Order of Y(SY) ! an attempt to obtain reasonable comparison values for
         SUBROUTINE EVALUATE X
         SUBROUTINE EVALUATE V(VN,WI)
         SUBROUTINE Relativistic ACCELERATIONS(ACC,ACCGR,Va,spina,dspin)
         subroutine Relativistic terms_not in use ! at the moment this is not used
         subroutine Relativistic terms!_ in use ! this is used for PN-term accelarations
         subroutine Reduce2cm(x,m,nb,cm)
         subroutine cross(a,b,c)
         subroutine gopu_SpinTerms(X,V,r,M1,m2,c,alpha,dv3,dalpha)
         subroutine Q2term(m,r,x,v,c,Q2,e,dvq) ! Quadrupole as C. Will advised
         SUBROUTINE Initial Stepsize(X,V,M,NB,ee,step) ! guesswork for initial stepsize
         subroutine elmnts ! two-body orbital elements (WITHOUT relativistic corrections)
         function atn2(s,c) ! atan 4 interval (0,2Pi) !  not  subroutine
         function oot(alfa,eta,zeta,q,e,sqaf) ! oot=pericentre time !  not  subroutine
         function g3(z) !  not  subroutine 
         SUBROUTINE CONSTANTS OF MOTION(ENE_NB,G,Alag) ! Newtonian constants of motion (i.e. no PN)
         SUBROUTINE FIND BINARIES(time)  ! this is a toy routine for finding binaries
         SUBROUTINE  WCMOTION(hs)  ! This routine evaluates the velocity jumps in leapfrog
         subroutine V_jump(WCj,spinj,cmvj,wttlj,WCi,spini,FCj,acc,dt,
         subroutine V_jACConly(WCj,CMVj,WTTLj,FC,acc,dt,
         subroutine Estimate Stepsize(dtime,step)!Stumpff-Weiss method to estimate the s-step
        subroutine gfunc(xb,al,g) ! G_n=xb^n c_n(al*xb^2)
        SUBROUTINE cfun(z,c)!Stumpff c-functions
        subroutine Xanom(m,r,eta,zet,beta,t,x,rx,g) ! Solving Kepler's equation
        function cdot(a,b) !  not  subroutine 
         subroutine  Non relativistic v_dependent perturbations(df) ! USER DEFINED ( to be programmed by the user)
        subroutine COORDINATE DEPENDENT PERTURBATIONS(ACC) ! USER DEFINED
