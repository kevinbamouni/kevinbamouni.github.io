---
layout: post
title:  "Monte carlo and variance reduction for option pricing in C++"
date:   2018-01-01 10:13:32 +0100
---
# Monte Carlo method (Implementation in C++)

## Monte Carlo method : Definition

"Monte Carlo methods (or Monte Carlo experiments) are a broad class of computational algorithms that rely on repeated random sampling to obtain numerical results. Their essential idea is using randomness to solve problems that might be deterministic in principle. They are often used in physical and mathematical problems and are most useful when it is difficult or impossible to use other approaches. Monte Carlo methods are mainly used in three distinct problem classes:[1] optimization, numerical integration, and generating draws from a probability distribution." : https://en.wikipedia.org/wiki/Monte_Carlo_method

## Monte Carlo method and variance reduction

Monte carlo method execution can be very long in computation time, so there are some methods called variance reduction method that help reduce computation time then accelerate the convergence of the method and reduce the confidence intervall.

## Call pricing with monte carlo

The price or premium of a call is his probable payoff discounted. So knowing the payoff of a call, the premium is :
$$ Call_t = max(S-K, 0) \times e^{-r(T-t)} $$

At t, it is impossible to know exactly what will be the payoff at T the maturity of the Call, because of the price of a underliying follow a stochastic process, So the payoff is expected. Then :

$$ Call_t = e^{-r(T-t)} \times E((S-K)^+) $$

To compute the call price by monte carlo

## C++ Code

The code below is c++ code that compute the premium of a call using monte carlo method and then use variance reduction method to

```cpp
#include <random>
#include <iostream>
#include <cmath>
#include <algorithm>


// fonctions declarations
double alea_uniform_dist();
double alea_normal_dist(const double& stddev);
double alea_normal_dist(const double& moydev, const double& stddev);
// fonction de calcul de moyenne
template<typename T, typename S>
T moyenne(S tab[], int& taille){
    T somme(0);
    for (int i(0);i<taille;i++){
        somme = somme + tab[i];
    }
    return somme/taille;
}

// fonction de calcul de variance sans biais
template<typename T, typename S>
T variance( S tab[],  int& taille){
    T variance_quad(0);
    for (int i(0);i<taille;i++){
        variance_quad = variance_quad + (tab[i]-moyenne<T,S>(tab, taille))*(tab[i]-moyenne<T,S>(tab, taille));
    }
    return variance_quad/(taille-1);
}

//*********************************************************************************************************
//************************************************MAIN*****************************************************
int main()
{

    // Print Application title
    std::cout << "EUROPEAN CALL OPTION PRICING WITH THE MONTE CARLO METHOD"<<std::endl;
    std::cout << "********************************************************"<<std::endl;

    // 1 PARAMETERS INITIALISATION
    // Parameters of the call option that we want to price
    double S0(100); // actual price of the underlying
    double K(100); // the strike of the call option
    double r(0.05); // risk free rate
    double T(1.0); // the maturity of the option
    double sigma(0.10); // the volatility of the underlying, for simplification we take it constant
    int N(252); // number of period per year
    int M(100000); // number of simulation;1000-1,18sec-4.3802 10000-11,649sec-4.37739; 100000-114,556
    double S[N+1]; S[0] = S0;// the path of the underlying, the fist element is the actual price of the underlying
    double dt(T/N); // time step of the underlying simulation
    double payoffs[M]; //
    double premium(0);
    double stddev(1.0);

    // print Input parameters
    std::cout << "THE CALL PARAMETERS :"<<std::endl;
    std::cout << "S0 = " << S0 <<std::endl;
    std::cout << "K = " << K <<std::endl;
    std::cout << "r = " << r <<std::endl;
    std::cout << "T = " << T <<std::endl;
    std::cout << "sigma = " << sigma <<std::endl;
    std::cout << "Monte carlo number of simulations = " << M <<std::endl;
    std::cout << "********************************************************"<<std::endl;
    std::cout << "REAL CALL PREMIUM COMPUTE WITH B&S: 6,80495 " <<std::endl;
    std::cout << "********************************************************"<<std::endl;

    // 2 Payoffs simulations loop
    for(int j(0);j<M;j++){

        // 3 path simulation loop
        for(int i(0);i<N+1;i++){
        S[i+1] = S[i]*exp((r-(0.5*sigma*sigma))*dt+sigma*sqrt(dt)*alea_normal_dist(0.0,stddev));
        }

        // 4 compute the sum of payoffs of the simulations
        payoffs[j] = std::max(S[N] - K,0.0);
    }

    // 5 Dicounted expected premium computation
    double moypayoff(moyenne<double,double>(payoffs,M)); // simulated payoff mean estimation
    premium = exp(-r*T)*(moypayoff); // Actualise la moyenne des payoff pour fournir le prix

    // estimation precisions details
    double pay_stddev(0); // standard deviation of the payoffs simulated
    pay_stddev = sqrt(variance<double,double>(payoffs,M)); // standard deviation of the payoffs simulated

    // print the premium value
    std::cout << "********************************************************"<<std::endl;
    std::cout << "THE SIMULATION DETAILS : "<<std::endl;
    std::cout << "The payoffs mean: "<< moypayoff << std::endl;
    std::cout << "The premium of the call option is : "<<premium << std::endl;
    //std::cout << "The payoffs std_deviation : "<< pay_stddev << std::endl;
    std::cout << "confidence interval of the mean estimation: [" << premium-2*(pay_stddev/sqrt(M)) << " ; "<< premium+2*(pay_stddev/sqrt(M)) << "]" << std::endl;
    std::cout << "The confidence interval size: "<< (premium+2*(pay_stddev/sqrt(M))) - (premium-2*(pay_stddev/sqrt(M))) << std::endl;


    // ****************************************************************************************************************************************
    // *************************************************** REDUCTION DE VARIANCE **************************************************************

    // ***************************************************** 1 VARIABLE ANTITETHIC *************************************************************

    // 2 Payoffs simulations loop
    // In this loop we replace half of the payoff already simulated with the antithetic variable

    for(int j(0);j<M;j++){

        // 3 path simulation loop
        for(int i(0);i<N+1;i++){
        S[i+1] = S[i]*exp((r-(0.5*sigma*sigma))*dt+sigma*sqrt(dt)*-alea_normal_dist(0.0, stddev)); // sim with antitethic variate -X of X
        }

        // 4 compute the sum of payoffs of the simulations
        payoffs[j] =  (payoffs[j] + std::max(S[N] - K,0.0))/2;
    }

    //double moypayoff_ant(moyenne<double,double>(payoffs,M)); // simulated payoff mean estimation with antithetic variates
    double moypayoff_rant((moyenne<double,double>(payoffs,M))); // moyenne des payoffs obtenus par variable antithétiques
    premium = exp(-r*T)*(moypayoff_rant); // Actualise la moyenne des payoff pour fournir le prix
    double pay_stddev_ant(0); // standard deviation of the payoffs simulated with antithetic variates
    pay_stddev_ant = sqrt(variance<double,double>(payoffs,M)); // standard deviation of the payoffs simulated

    // print the premium value
    std::cout << "********************************************************"<<std::endl;
    std::cout << "THE ANTITHETIC VARIATE OPTIMISATION SIMULATION DETAILS : "<<std::endl;
    std::cout << "The payoffs mean: "<< moypayoff_rant << std::endl;
    std::cout << "The premium of the call option is : "<<premium << std::endl;
    //std::cout << "The payoffs std_deviation : "<< pay_stddev_ant << std::endl;
    std::cout << "confidence interval of the mean estimation: [" << premium-2*(pay_stddev_ant/sqrt(M)) << " ; "<< premium+2*(pay_stddev_ant/sqrt(M)) << "]" << std::endl;
    std::cout << "The confidence interval size: "<< (premium+2*(pay_stddev_ant/sqrt(M)))-(premium-2*(pay_stddev_ant/sqrt(M))) << std::endl;


    // ****************************************************************************************************************************************
    // ******************************************************** 2 CONTROL VARIATE TECHNIQUE ******************************************************
    // Simulation of put payoffs to valuate the call

    for(int j(0);j<M;j++){

        // 3 path simulation loop
        for(int i(0);i<N+1;i++){
        S[i+1] = S[i]*exp((r-(0.5*sigma*sigma))*dt+sigma*sqrt(dt)*alea_normal_dist(0.0,stddev));
        }

        // 4 compute the sum of payoffs of the simulations
        payoffs[j] = exp(r*T)*S0 - K + std::max(K - S[N] ,0.0); // PUT PAYOFF
    }

    double callpayoff(0);
    callpayoff = moyenne<double,double>(payoffs,M); // CALL VALUE DEDUCES FROM CALL-PUTT PARITY FORMULA
    premium = exp(-r*T)*(callpayoff); // Actualise la moyenne des payoff pour fournir le prix
    double pay_stddev_varcontr(0); // standard deviation of the payoffs simulated with antithetic variates
    pay_stddev_varcontr = sqrt(variance<double,double>(payoffs,M)); // standard deviation of the payoffs simulated

    std::cout << "********************************************************"<<std::endl;
    std::cout << "THE CONTROL VARIATE OPTIMISATION DETAILS : "<<std::endl;
    std::cout << "The payoffs mean: "<< callpayoff << std::endl;
    std::cout << "The premium of the call option is : "<< premium << std::endl;
    //std::cout << "The payoffs std_deviation : "<< pay_stddev_varcontr << std::endl;
    std::cout << "confidence interval of the mean estimation: [" << premium-2*(pay_stddev_varcontr/sqrt(M)) << " ; "<< premium+2*(pay_stddev_varcontr/sqrt(M)) << "]" << std::endl;
    std::cout << "The confidence interval size: "<< (premium+2*(pay_stddev_varcontr/sqrt(M)))-(premium-2*(pay_stddev_varcontr/sqrt(M))) << std::endl;

    // ****************************************************************************************************************************************
    // ******************************************************** 3 FONCTION D'IMPORTANCE ******************************************************
    // Simulation of put payoffs to valuate the call *
    // CALL TRES DEHORS DE LA MONNAIE

    for(int j(0);j<M;j++){

        // 3 path simulation loop
        for(int i(0);i<N+1;i++){
        S[i+1] = S[i]*exp((r-(0.5*sigma*sigma))*dt+sigma*sqrt(dt)*alea_normal_dist(-0.01, stddev));
        }

        // 4 compute the sum of payoffs of the simulations
        payoffs[j] = std::max(S[N] - K,0.0);
    }

    // 5 Dicounted expected premium computation
    moypayoff = moyenne<double,double>(payoffs,M); // simulated payoff mean estimation
    premium = exp(-r*T)*(moypayoff); // Actualise la moyenne des payoff pour fournir le prix

    // estimation precisions details
    pay_stddev = sqrt(variance<double,double>(payoffs,M)); // standard deviation of the payoffs simulated

    // print the premium value
    std::cout << "********************************************************"<<std::endl;
    std::cout << "THE IMPORTANCE FUNCTION OPTIMISATION : "<<std::endl;
    std::cout << "The payoffs mean: "<< moypayoff << std::endl;
    std::cout << "The premium of the call option is : "<<premium << std::endl;
    //std::cout << "The payoffs std_deviation : "<< pay_stddev << std::endl;
    std::cout << "confidence interval of the mean estimation: [" << premium-2*(pay_stddev/sqrt(M)) << " ; "<< premium+2*(pay_stddev/sqrt(M)) << "]" << std::endl;
    std::cout << "The confidence interval size: "<< (premium+2*(pay_stddev/sqrt(M))) - (premium-2*(pay_stddev/sqrt(M))) << std::endl;

    //verifying that the program finish
    return 0; //verifying that the program finish
}

// fonctions code

// Generate a number with follow a uniform distribution
// : http://en.cppreference.com/w/cpp/numeric/random/uniform_real_distribution
double alea_uniform_dist(){
    // generate random uniform numbers by c++11
    std::random_device rd;  //Will be used to obtain a seed for the random number engine
    std::mt19937 gen(rd()); //Standard mersenne_twister_engine seeded with rd()
    std::uniform_real_distribution<> dis(0.0, 1.0);
    //Use dis to transform the random unsigned int generated by gen into a double in [0, 1)
    return dis(gen); //Each call to dis(gen) generates a new random double
}


// Generate a number with follow a gaussian distribution
// : http://en.cppreference.com/w/cpp/numeric/random/normal_distribution
double alea_normal_dist(const double& stddev){
    std::random_device rd;  //Will be used to obtain a seed for the random number engine
    std::mt19937 gen(rd()); //Standard mersenne_twister_engine seeded with rd()
    std::normal_distribution<> dis(0.0, stddev);
    return dis(gen);
}

double alea_normal_dist(const double& moydev, const double& stddev){
    std::random_device rd;  //Will be used to obtain a seed for the random number engine
    std::mt19937 gen(rd()); //Standard mersenne_twister_engine seeded with rd()
    std::normal_distribution<> dis(moydev, stddev);
    return dis(gen);
}

```

Here is below the result of the execution of tje above c++ code.

So the code implement a monte carlo method for call pricing. The datails of the call are listed below. Using the call formula of
black and scholes i get the exact price of the call and then i perform a monte carlo method to try to find the same price as the call formula. I perform a classic monte carlo method, the three methods of varaiance reduction that are :
- Antithetic variate
- Control variate
- Importance funtion

for each implementation i also compute the confidence intervall of the approximation of the call premium. That confidence intervall uses the central limit theorem to show the convergence's speed of the method and the error intervall. So less the confidence intervall is large better the convergence's speed is. It is also important that the exact premium value be in the confidence intervall.

For the test, i compute a call value of 6,80495 obtain with B&S, then i implement monte carlo and variance reduction method on the same call caracteristics to compare accuracy. In order from the less precise to the more precise :

IMPORTANCE FUNCTION < ANTITHETIC VARIATE < CLASSIC MONTE CARLO < CONTROL VARIABLE

The importance funtion variance reduction method is a method specialy adapted for "in the money" and "out the money" Call option.

```
EUROPEAN CALL OPTION PRICING WITH THE MONTE CARLO METHOD
********************************************************
THE CALL PARAMETERS :
S0 = 100
K = 100
r = 0.05
T = 1
sigma = 0.1
Monte carlo number of simulations = 100000
********************************************************
REAL CALL PREMIUM COMPUTE WITH B&S: 6,80495
********************************************************
********************************************************
THE SIMULATION DETAILS :
The payoffs mean: 7.13361
The premium of the call option is : 6.7857
confidence interval of the mean estimation: [6.73429 ; 6.83711]
The confidence interval size: 0.102827
********************************************************
THE ANTITHETIC VARIABLE OPTIMISATION SIMULATION DETAILS :
The payoffs mean: 7.12914
The premium of the call option is : 6.78145
confidence interval of the mean estimation: [6.74515 ; 6.81774]
The confidence interval size: 0.0725883
********************************************************
THE CONTROL VARIABLE OPTIMISATION DETAILS :
The payoffs mean: 7.15181
The premium of the call option is : 6.80301
confidence interval of the mean estimation: [6.77777 ; 6.82825]
The confidence interval size: 0.050481
********************************************************
THE IMPORTANCE FUNCTION OPTIMISATION :
The payoffs mean: 6.01899
The premium of the call option is : 5.72544
confidence interval of the mean estimation: [5.67772 ; 5.77316]
The confidence interval size: 0.0954378

Process returned 0 (0x0)   execution time : 567.752 s
Press ENTER to continue.
```

### References

- Advanced quantitative finance with c++
- Wikipedia
- and some master degree lectures that i found on internet, thanks google.
