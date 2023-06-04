The code is not completely done and has not been properly tested yet.

I have uploaded this project for educational purpuses as i think it is a quite intuitive way of solving the approximate pattern matching problem. I have not heard of or seen this algorithm before so i guess i can name it PAM (Pigeonhole approximative matching). Feel free correcting or optimizing parts. 

The algorithm is build on following principles:

Allowing d mismatches we use the pigeonhole principle to devide our sustring into d+1 pieces. One of these pices can be found using exact pattern matching (which is a much easier problem to solve).
These pieces are then search using binary search in a suffix array constructed from the main string (which in this implementation is created by prefix doubling). 







