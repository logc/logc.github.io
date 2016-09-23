    Title: Wordcount in C++
    Date: 2014-01-21T14:52:26Z+0100
    Tags: C++

This is the standard example for MapReduce applications, counting the words in
a file. Note that this snippet is not threaded nor distributed. It is just a
curiosity: how to implement the same in C++, using the STL to avoid having any
`for` loop!

<!-- more -->

```c++
#include <fstream>
#include <iostream>
#include <numeric>
#include <sstream>
#include <string>
#include <vector>

using std::pair;
using std::string;
using std::vector;
using std::istringstream;
using std::istream_iterator;


vector<string> split_at_whitespace(string line)
{
    istringstream iss(line);
    vector<string> tokens{
        istream_iterator<string>{iss},
        istream_iterator<string>{}};
    return tokens;
}

vector< pair<string, int> > map(string line)
{
    vector<string> tokens = split_at_whitespace(line);
    vector< pair<string, int> > current_outputs(tokens.size());
    std::transform(
        tokens.begin(), tokens.end(), current_outputs.begin(),
        [] (const string output) {
            return std::make_pair(output, 1);
        });
    return current_outputs;
}

int reduce(vector< pair<string, int> > outputs)
{
    return std::accumulate(
        outputs.begin(), outputs.end(), 0.0,
        [] (int sum, const pair<string, int> output) {
                return sum + output.second;
            });
}

int main(int argc, const char *argv[])
{
    if (argc < 2) {
        std::cout << "Usage: " << argv[0] << " infile" << std::endl;
        return 1;
    }

    string filename = argv[1];
    std::ifstream infile(filename);
    string line;
    vector< pair<string, int> > occurences;
    while (std::getline(infile, line)) {
        auto line_occurences = map(line);
        occurences.insert(
            occurences.end(), line_occurences.begin(), line_occurences.end());
    }
    int n = reduce(occurences);
    std::cout << "File " << filename << " holds " << n << " words" << std::endl;
    return 0;
}
```

Compile and run:

```console
$ cat << EOF > sample.txt
> This is a first line
> This is a second line
> EOF

$ c++ main.cpp -o wordcount -std=c++11 && ./wordcount sample.txt
File sample.txt holds 10 words
```
