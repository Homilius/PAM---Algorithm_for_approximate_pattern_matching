The code is incomplete and has not undergone thorough testing yet (put the principle works well). I will fix this when i have time.

I have uploaded this project to share an idea and because I believe it provides an intuitive approach to solving and understanding the approximate pattern matching problem. I haven't come across or seen this algorithm before, so I propose naming it PAM (Pigeonhole Approximative Matching).

The algorithm is built on the following principles:

By allowing up to 'd' mismatches, we apply the pigeonhole principle and divide our substring into 'd+1' pieces. We can find one of these pieces (if a match exists) using exact pattern matching, which is a relatively simpler problem to solve. These pieces are then searched using binary search in a suffix array constructed from the reference string. In this implementation, the suffix array is created through prefix doubling. Once we obtain an exact match of one of these pieces, we can backtrack to the position that corresponds to the beginning of our pattern (note that this position is in relation to the reference string but potential indels are in relation to the pattern thus we will have to do some additional steps if we want the exact position with respect to the reference string). These steps provide us with all candidate positions that warrant further investigation.
Next, we perform a series of local alignments using these intervals (-d in the beginning +d in the end to capture potential indels; not counting mismatches in the ends). Finally, we select the best alignment (within d edits) and return the matching position along with its corresponding cigar string.

Some of these steps have a worst-case running time that can be slow. However, the main idea here is that the algorithm incurs very little overheads and it is unlikely that we will need to perform a significant amount of extra work. For example, if we divide a pattern of size, say, 100 into four pieces of size 25 and obtain a match, it is very probable that this match represents a genuine position. This is because a match of size 25 is relatively unlikely to occur multiple times by chance alone, at least in a genome. Therefore, while the worst-case running time may not be ideal, we are somewhat protected as long as our pattern has a certain length. 

I am convinced that this algorithm has the potential to run quite efficiently with the right implementation (rigth now a lot of stuff can be optimized). Please feel free to correct or optimize any parts and shoot me an email antonhomilius@hotmail.com.


############################################################
# Librarys:


import re
import random
import numpy as np

############################################################
# Functions:


def edits_to_cigar(edits: str):
    def split_blocks(x: str):
        return [m[0] for m in re.findall(r"((.)\2*)", x)]
    element_list = split_blocks(edits)
    cigar=''
    for a in element_list:
        short = '{}{}'.format(len(a),a[0])
        cigar+= short
    return cigar


def get_edits(p: str, q: str):
    assert len(p) == len(q)
    edits = ''
    for i in range(len(p)):
        if p[i] != '-' and q[i] != '-':
            edits += 'M'
        if p[i] == '-' and q[i] != '-':
            edits += 'I'
        if p[i] != '-' and q[i] == '-':
            edits += 'D'
    if len(p) == 0 and len(q) == 0:
        p_out = ''
        q_out = ''
        edits = ''
    else:
        p_out = p.replace('-','')
        q_out = q.replace('-','')
        edits = edits
    return p_out, q_out , edits


def SuffixArray(string):
    if string == '' or string == None:
        return None
    string += '$'
    index = {v: i for i, v in enumerate(sorted(set(string)))}
    string = [index[v] for v in string]
    rank_list = list(range(len(string)))
    SA = [None]*len(string)
    tuple_list = SA[:]
    M,j = 0,1
    while M < len(string)-1:
        for i in range(len(string)):
            i_j = i+j
            if i_j < len(string):
                key = (string[i], string[i_j])
            else: 
                key = (string[i], string[-1])
            tuple_list[i] = key
        j*=2
        keys = sorted(set(tuple_list))
        ranks = dict(zip(keys, rank_list))
        string = [ranks[tuple_list[i]] for i in range(len(string))]
        M = max(string)
    for i in rank_list:
        SA[string[i]] = i
    return SA


def binary_search(SA, string, pattern):
    if pattern == '' or pattern == None:
        return None
    string+='$'
    SA_positions = []
    # Binary search:
    hi, lo = len(SA), 0, 
    match = None
    B = 0
    while lo < hi and B == 0:
        mid = (lo + hi) // 2
        count = 0
        for i in range(count, len(pattern)):
            if pattern[i] == string[SA[mid]+i]:
                count+=1
            if count == len(pattern):
                match = mid    
                SA_positions.append(SA[match])
                B=1
            elif pattern[i] > string[SA[mid]+i]:
                lo = mid + 1
                break
            elif pattern[i] < string[SA[mid]+i]:
                hi = mid
                break
    # Scan up/down from match pos:
    if match != None:
        k=1
        while match-k >= 0 and string[SA[match-k]:SA[match-k]+len(pattern)] == pattern:
            SA_positions.append(SA[match-k])
            k+=1
        k=1
        while match+k < len(string) and string[SA[match+k]:SA[match+k]+len(pattern)] == pattern:
            SA_positions.append(SA[match+k])
            k+=1  
    return SA_positions


def local_matrix(seq1, seq2, d):

    def create_matrix(seq1, seq2):
        rows = len(seq1) + 1
        cols = len(seq2) + 1
        matrix = np.zeros((rows, cols), dtype=int)
        for i in range(1, rows):
            for j in range(1, cols):
                matrix[i, 0] = i
                matrix[0, j] = j
        for col in range(1, cols):
            for row in range(1, rows):
                cost = 0 if seq1[row - 1] == seq2[col - 1] else 1
                matrix[row, col] = min(matrix[row - 1, col] + 1,
                                       matrix[row, col - 1] + 1,
                                       matrix[row - 1, col - 1] + cost)
        return matrix

    def check_dist(seq1, seq2, matrix, d):
        row, col = len(seq1), len(seq2)
        # Allowing mismatches:
        last_row = matrix[len(seq1),:]
        last_row_min_d_pos = len(last_row)-(d)+np.argmin(last_row[-(d):])
        last_col = matrix[:,len(seq2)]
        last_col_min_d_pos = len(last_col)-(d)+np.argmin(last_col[-(d):])
        t0 = matrix[last_col_min_d_pos, len(seq2)]      
        t1 = matrix[len(seq1), last_row_min_d_pos]
        edit_distance = min(t0,t1)
        if edit_distance <= d:
            return edit_distance
        else: 
            return None

    matrix = create_matrix(seq1, seq2)
    edit_distance = check_dist(seq1, seq2, matrix, d)
    if edit_distance != None:
        return (edit_distance, matrix)
    else: 
        return None


def backtrace(seq1, seq2, matrix):
        seq1, seq2 = list(seq1), list(seq2)
        aligned1, aligned2 = [], []
        row, col = len(seq1), len(seq2)
        while True:
            cost = 0 if seq1[row - 1] == seq2[col - 1] else 1
            cur = matrix[row, col]
            vertical = matrix[row-1, col]
            diagonal = matrix[row - 1, col - 1]
            horizontal = matrix[row, col - 1]
            if cur == diagonal + cost:
                aligned1 += [seq1[row - 1]]
                aligned2 += [seq2[col - 1]]
                row, col = row - 1, col - 1
            else:
                if cur == vertical + 1:
                    aligned1 += [seq1[row - 1]]
                    aligned2 += ["-"]
                    row, col = row - 1, col
                elif cur == horizontal + 1:
                    aligned1 += ["-"]
                    aligned2 += [seq2[col - 1]]
                    row, col = row, col - 1
            if row == 0 and col == 0:
                return ''.join(aligned1[::-1]), ''.join(aligned2[::-1])


def approx_positions(string, pattern, SA, d):
    n_segments = d+1
    segment_size = int(round(len(pattern)/n_segments))
    approx_pos = set()
    approx_trimmed = set()
    
    # Generate all possible start positions given d+1 possible exact matching segments:
    exp_starts = set()
    for s in range(n_segments):
        segment_start = s*segment_size
        segment_end = segment_start + segment_size
        if segment_end > len(pattern): segment_end = len(pattern)
        matches = binary_search(SA, string, pattern[segment_start:segment_end])
        if matches != None:
            for m in matches:
                exp_start = m-segment_start
                if exp_start >= 0-d and exp_start <= len(string)+d:
                    exp_starts.add(exp_start)
    
    # Generate all possible pattern intervals:
    possible_intervals = set()
    for val in exp_starts:
        for flex in range(-d,d+1,1):
            flex_start = val + flex 
            flex_end = val + len(pattern) + (d-abs(flex))
            if flex_start >= 0 and flex_end <= len(string):
                possible_intervals.add((flex_start, flex_end))
            else:
                if flex_start >= 0 and flex_end > len(string):
                    possible_intervals.add((flex_start, len(string)))
                if flex_start < 0 and flex_end <= len(string):
                    possible_intervals.add((0, flex_end))
    possible_intervals = list(possible_intervals)

    # Get all matrices with best-fit <= d:
    for tup in possible_intervals:
        seq, d_max = string[tup[0]:tup[1]], d
        if 0 <= abs(len(seq) - len(pattern)) <= d_max:
            matrix_inf = local_matrix(pattern, seq, d_max)
            if matrix_inf != None:
                if matrix_inf[0] <= d_max:
                    # print('Matrix_and_pos: \n', matrix_inf[1], tup[0], '\n n_edits:', matrix_inf[0])
                    
                    # Get best alignment:
                    seq1, seq2 = list(pattern), list(seq)
                    matrix = matrix_inf[1]  
                    alignment = backtrace(seq1, seq2, matrix)
                    approx_pos.add((tup[0], alignment[0], alignment[1]))
                                        
    for alignment in approx_pos:
        start_gaps = 0
        j = 0
        while alignment[1][j] == '-':
            start_gaps+=1
            j+=1
        end_gaps = 0
        j = 0
        while alignment[1][::-1][j] == '-':
            end_gaps+=1
            j+=1
        ends_gaps = (start_gaps+end_gaps)
        al1_indels = alignment[1].count('-') - ends_gaps
        pos = alignment[0]+start_gaps
        if len(alignment[1])-ends_gaps >= len(pattern) + al1_indels:
            mm = 0
            for i in range(len(alignment[1])-ends_gaps):
                if alignment[1][i+start_gaps] != alignment[2][i+start_gaps]:
                    mm+=1
                if mm > d: break
            if mm <= d:
                al1 = ''.join(alignment[1][0+start_gaps:len(alignment[1])-end_gaps])
                al2 = ''.join(alignment[2][0+start_gaps:len(alignment[2])-end_gaps])
                edits = get_edits(al2,al1)
                cigar = edits_to_cigar(edits[2])
                approx_trimmed.add((pos,cigar,mm))
    return list(approx_trimmed)

############################################################
# Usage:
string = 'AACGATGGTCGGCGTATTAACGACTCCAAACCCAACCATCGAGTATTGAGGTTACTACAGAAAAATAAAAACGACAAACAATGAAGCTGGGATTTTTGTAGCTTGCAACCTTCATAACCGGCAGCTAGCGTCTAGTGAAAAGGCTCCATTGGCAGGTGTGAATCAACAAAGAATTGGG'
pattern = 'GTACGAAAATTAAACGACAGACAATGAAGCTTGGGA'
SA = SuffixArray(string)
print(approx_positions(string, pattern, SA, 10)) 

############################################################





