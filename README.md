# AiSD_set3_a2


struct ArrayGenerator {

    const int MAX_SIZE = 100000;
    const int MIN_VAL = 0;
    const int MAX_VAL = 6000;
    const double NEARLY_SORTED_SWAP_RATIO = 0.05;

    std::vector<int> randomArray;
    std::vector<int> reversedArray;
    std::vector<int> nearlySortedArray;

    ArrayGenerator() {
        randomArray.resize(MAX_SIZE);
        reversedArray.resize(MAX_SIZE);
        nearlySortedArray.resize(MAX_SIZE);

        std::random_device rd;
        std::mt19937 gen(rd());
        std::uniform_int_distribution<> dis(MIN_VAL, MAX_VAL);

        for (int i = 0; i < MAX_SIZE; ++i) {
            randomArray[i] = dis(gen);
        }

        for (int i = 0; i < MAX_SIZE; ++i) {
            reversedArray[i] = dis(gen);
        }
        std::sort(reversedArray.begin(), reversedArray.end());
        std::reverse(reversedArray.begin(), reversedArray.end());

        std::iota(nearlySortedArray.begin(), nearlySortedArray.end(), MIN_VAL);
        int swaps = static_cast<int>(MAX_SIZE * NEARLY_SORTED_SWAP_RATIO / 2);
        std::uniform_int_distribution<> dis_index(0, MAX_SIZE - 1);
        for (int i = 0; i < swaps; ++i) {
            int ind1 = dis_index(gen);
            int ind2 = dis_index(gen);
            if (ind1 != ind2) {
                std::swap(nearlySortedArray[ind1], nearlySortedArray[ind2]);
            }
        }
    }
    std::vector<int> getSubArray(const std::vector<int>& baseArray, int size) {
        return std::vector<int>(baseArray.begin(), baseArray.begin() + size);
    }
};


class SortTester {

public:
    std::vector<int> thresholds = {10, 40, 80};
    const int START_SIZE = 90000;
    const int END_SIZE = 100000;
    const int STEP_SIZE = 100;
    const int NUM_RUNS = 7;
    ArrayGenerator generator;
    std::map<std::string, std::map<std::string, std::map<int, double>>> results;

    void runFullTest() {
        test("Random", generator.randomArray);
        test("Reversed", generator.reversedArray);
        test("NearlySorted", generator.nearlySortedArray);
        saveResultsToCSV("results.csv");
    }

private:
    void test(const std::string& type, const std::vector<int>& arr) {
        for (int size = START_SIZE; size <= END_SIZE; size += STEP_SIZE) {
            results[type]["merge"][size] = timeMergeSort(arr, size);
            for (auto threshold : thresholds) {
                results[type]["merge+insertion threshold = " + std::to_string(threshold)][size] = timeGybridMergeSort(arr, size, threshold);
            }
        }
    }

    double timeMergeSort(const std::vector<int>& arr, int size) {
        std::vector<long long> times;
        for (int i = 0; i < NUM_RUNS; i++) {
            std::vector<int> testArr = generator.getSubArray(arr, size);
            auto start = std::chrono::high_resolution_clock::now();
            mergeSort(testArr, 0, size - 1);
            auto elapsed = std::chrono::high_resolution_clock::now() - start;
            long long nsec = std::chrono::duration_cast<std::chrono::nanoseconds>(elapsed).count();
            times.push_back(nsec);
        }
        std::sort(times.begin(), times.end());
        long long median_ns = times[times.size() / 2];
        return static_cast<double>(median_ns) / 1000000.0;
    }

    double timeGybridMergeSort(const std::vector<int>& arr, int size, int threshold) {
        std::vector<long long> times;
        for (int i = 0; i < NUM_RUNS; i++) {
            std::vector<int> testArr = generator.getSubArray(arr, size);
            auto start = std::chrono::high_resolution_clock::now();
            gybridMergeSort(testArr, 0, size - 1, threshold);
            auto elapsed = std::chrono::high_resolution_clock::now() - start;
            long long nsec = std::chrono::duration_cast<std::chrono::nanoseconds>(elapsed).count();
            times.push_back(nsec);
        }
        std::sort(times.begin(), times.end());
        long long median_ns = times[times.size() / 2];
        return static_cast<double>(median_ns) / 1000000.0;
    }

    void saveResultsToCSV(const std::string& filename) {
        std::ofstream outFile(filename);
        outFile << "DataType;SortType;Threshold;Size;Time_ms\n";
        for (const auto& typePair : results) {
            const std::string& typeName = typePair.first;
            for (const auto& algoPair : typePair.second) {
                const std::string& algoName = algoPair.first;
                int threshold = 0;
                if (algoName.find("merge+insertion") != std::string::npos) {
                    threshold = std::stoi(algoName.substr(algoName.find("= ") + 2));
                }
                for (const auto& dataPair : algoPair.second) {
                    int size = dataPair.first;
                    double time = dataPair.second;
                    outFile << typeName << ";"
                            << (threshold > 0 ? "Gybrid" : "Standard") << ";"
                            << threshold << ";"
                            << size << ";"
                            << time << "\n";
                }
            }
        }
        outFile.close();
    }
};
