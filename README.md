# PAM (Pigeonhole Approximative Matching)

## Overview
This project presents an intuitive approach to solving and understanding the approximate pattern matching problem. The algorithm, which I propose naming **Pigeonhole Approximative Matching (PAM)**, leverages the **pigeonhole principle** to efficiently identify approximate matches in a reference string.

### Status
The implementation is **incomplete** and has **not undergone thorough testing** yet. The core principle works well, but further optimizations and debugging are needed. I will improve and refine the implementation when I have time.

## Algorithm Principle
The PAM algorithm is built upon the following ideas:

1. **Pigeonhole Principle & Substring Division**
   - Given a pattern with up to `d` mismatches, we split it into `d+1` segments.
   - If a match exists, at least one of these segments must match exactly with the reference string.

2. **Exact Matching Using a Suffix Array**
   - The reference string is preprocessed into a **suffix array** using the **prefix-doubling method**.
   - We perform **binary search** on the suffix array to locate exact matches for one of the pattern's segments.

3. **Candidate Position Identification**
   - After finding an exact match for one of the segments, we backtrack to estimate the starting position of the full pattern.
   - This step accounts for potential insertions and deletions (indels), ensuring alignment with the reference string.

4. **Local Alignment & Best Match Selection**
   - We perform **local alignments** within an interval expanded by `-d` at the start and `+d` at the end.
   - This ensures we capture potential indels without counting mismatches at the boundaries.
   - The best alignment (within `d` edits) is selected, and we return its **matching position** along with the corresponding **CIGAR string**.

## Advantages & Considerations
- **Minimal Overhead:** The algorithm reduces unnecessary computations by focusing only on the most promising candidate positions.
- **Practical Efficiency:** Although the worst-case time complexity may not be optimal, real-world scenarios (e.g., genomic sequences) offer protection since a **random 25-character match is unlikely to occur multiple times**.
- **Optimization Potential:** The current implementation can be significantly improved in terms of speed and memory efficiency.

## Contributing
I believe this algorithm has the potential to be highly efficient with the right optimizations. Contributions and improvements are welcome! If you have any suggestions or optimizations, feel free to create a pull request or contact me at:

📧 **antonhomilius@hotmail.com**

---

### Disclaimer
This project is still a work in progress. Expect further refinements and enhancements in future updates.
