# Blockchain meets Biometrics: Concepts, Application to Template Protection, and Trends

```diff 
+ This repository contains the source code for the smart contracts deployed on the Ethereum blockchain platform.
```

This work is part of an ongoing research based on the integration of biometrics and blockchain technologies [BLOCKCHAIN_2019] [ArXiv_2020]. In this last paper [ArXiv_2020], we experimentally study the key tradeoffs involved in that integration, namely, latency, processing time, economic cost, and biometric performance. 

# ON-CHAIN FACE RECOGNITION MATCHING

In addition to the analysis of the economic costs and performance of biometric templates protection schemes included in this article, another major contribution is the **implementation of some native face recognition schemes in a blockchain**. Specifically, they are designed for face biometrics, and based on **Euclidean (unprotected case) and Hamming distances (protected cases)**.

***Euclidean distance***: The implementation of this operation as a smart contract is specially challenging, due to the lack of native support for floating point arithmetic. To overcome this limitation, we represent decimal numbers as integers (i.e.: 1.12 as 112) and use the Newton-Raphson method to obtain the nth-root of d by solving the equation x^n - d = 0 iteratively.

In this way, we can also adjust the cost of the operation according to the required precision. The results show that the cost of the calculation of a square root is 23,209 gas units, and 31,304 (around 0.000031 ETH or $0.00527 for the current ether price, March 2020) for the calculation of a complete matching operation.

At first glance it may seem a small value, but in a real biometric system, the matching operation must be repeated hundreds or thousands of times. For example, for a scenario with 10,000 users, the total cost for the matching operation (the authentication of each user in the system) would be slightly greater than $50. Possible solutions and improvements will be object of future research, as discussed inside the article [ArXiv_2020].

***Hamming distance***: On the other hand, Hamming distance is significantly easier to implement and, it is virtually free in economic terms.

Every operation implemented in a blockchain must be optimized as much as possible. In this case we use an algorithm whose running time depends on the number of ones present in the binary form of the given number, instead of just traversing and counting the number of ones, obtaining a much better performance.

In contrast to the Euclidean distance, this distance calculation is simpler, and only uses bitwise operations. In addition, it does not store data into the blockchain and, as a result, as a read-only operation has no execution costs. This is especially important in a biometric scenario, where a large number of users can be involved in matching operations. 

## REFERENCES

[BLOCKCHAIN_2019] O. Delgado-Mohatar, J. Fierrez, R. Tolosana and R. Vera-Rodriguez, "Blockchain and Biometrics: A First Look into Opportunities and Challenges", in Proc. International Congress on Blockchain and Applications, 2019.

[ArXiv_2020] O. Delgado-Mohatar, J. Fierrez, R. Tolosana and R. Vera-Rodriguez, "Blockchain meets Biometrics: Concepts, Application to Template Protection, and Trends”, arXiv preprint arXiv:2003.09262, 2020.

# SMART CONTRACT SOURCE CODE

``` 
pragma solidity ^0.5.11;
pragma experimental ABIEncoderV2;

contract MatchingOperation {
    
    // Calculates the Euclidean distance
    // The threshold value must be codified as an integer, 
    // with the appropiate precision, i.e.: 1256 for 1.256
    function templateMatching(uint threshold, uint _dp,
                         uint[] memory baseArray, uint[] memory userArray) public view returns(bool) {
        uint distance = 0;
        
        // Calculate the Euclidean distance between the two vectors
        for (uint i = 0; i < array1.length; i++){
            distance += (baseArray[i] - userArray[i])**2;
        }
        
        return nthRoot(distance, 2, _dp, 1000) <= threshold;
    }
    
    // Calculates a^(1/n) to dp decimal places
    // maxIts bounds the number of iterations performed
    function nthRoot(uint _a, uint _n, uint _dp, uint _maxIts) pure public returns(uint) {
        assert (_n > 1);

        // We actually do (a * (10 ^ ((dp + 1) * n))) ^ (1/n) 
        // to turn everything into integer calcs.
        uint one = 10 ** (1 + _dp);
        uint a0 = one ** _n * _a;
        // Initial guess: 1.0
        uint xNew = one;
        uint x = 0;
        uint t0 = 0;
        
        uint iter = 0;
        while (xNew != x && iter < _maxIts) {
            x = xNew;
            t0 = x ** (_n - 1);
            if (x * t0 > a0) {
                xNew = x - (x - a0 / t0) / _n;
            } else {
                xNew = x + (a0 / t0 - x) / _n;
            }
            ++iter;
        }

        // Round to nearest in the last dp.
        return (xNew + 5) / 10;
    }     
    
    function HammingDistance(int a, int b) pure public returns(int) {

        // Get A XOR B...
        int c = a ^ b;
        
        // and now count the number of ones
        int count=0;
        while (c!=0)
        {
            // This works because if a number is power of 2, 
            // then it has only one 1 in its binary representation.
            c = c & (c-1);
            count++;
        }
        
        return count;
    }
}

```


