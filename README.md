The code is not completely done and has not been properly tested yet.

I have uploaded this project for educational purpuses as i think it is a quite intuitive way of solving the approximate pattern matching problem. I have not heard of or seen this algorithm before so i guess i can name it PAM (Pigeonhole approximative matching). Feel free correcting or optimizing parts. 

The algorithm is build on following principles:

Allowing d mismatches we use the pigeonhole principle to devide our sustring into d+1 pieces. One of these pices can be found using exact pattern matching (which is a much easier problem to solve).
These pieces are then search using binary search in a suffix array constructed from the reference string (which in this implementation is created by prefix doubling). 
When we get an exact match of one of these pieces we can then go from there back to the position that should correspond to the beginning of our substring (note this position is with respect to the substring not the regerence string!).
So above steps will give us all candidate positions that could be interesting to investigate further. 
Now we do a simple local alignment. From the corresponding local alignment matrix we select the smallest value within 2*d entries away from last entry; last row and col). We do 2*d because one goes to actual edits + one d since we need to generate a flexible interval to find all possible matches within d edits (since the start position is with respect to the pattern and not the reference string and thus we need to takepossible indels into account).






