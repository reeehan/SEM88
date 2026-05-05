#include <iostream>
#include <queue>
#include <unordered_map>
using namespace std;

// Node structure
struct Node {
    char ch;
    int freq;
    Node *left, *right;

    Node(char c, int f) {
        ch = c;
        freq = f;
        left = right = NULL;
    }
};

// Comparator for priority queue
struct compare {
    bool operator()(Node* l, Node* r) {
        return l->freq > r->freq;
    }
};

// Generate Huffman Codes
void generateCodes(Node* root, string code,
                   unordered_map<char, string>& huffmanCode) {
    if (!root) return;

    if (!root->left && !root->right)
        huffmanCode[root->ch] = code;

    generateCodes(root->left, code + "0", huffmanCode);
    generateCodes(root->right, code + "1", huffmanCode);
}

int main() {
    string text;
    cout << "Enter string: ";
    cin >> text;

    // Count frequency
    unordered_map<char, int> freq;
    for (char ch : text)
        freq[ch]++;

    // Min heap
    priority_queue<Node*, vector<Node*>, compare> pq;

    for (auto pair : freq)
        pq.push(new Node(pair.first, pair.second));

    // Build Huffman Tree
    while (pq.size() != 1) {
        Node *left = pq.top(); pq.pop();
        Node *right = pq.top(); pq.pop();

        Node *sum = new Node('$', left->freq + right->freq);
        sum->left = left;
        sum->right = right;

        pq.push(sum);
    }

    Node* root = pq.top();

    unordered_map<char, string> huffmanCode;
    generateCodes(root, "", huffmanCode);

    // Print codes
    cout << "\nHuffman Codes:\n";
    for (auto pair : huffmanCode)
        cout << pair.first << " : " << pair.second << endl;

    // Encode string
    string encoded = "";
    for (char ch : text)
        encoded += huffmanCode[ch];

    cout << "\nEncoded String:\n" << encoded << endl;

    return 0;
}