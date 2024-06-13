<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Waste Bin Allocation</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
            background-color: #f0f9ff; /* Light blue-green background */
            color: #333; /* Dark gray text color */
        }
        .container {
            max-width: 800px;
            margin: auto;
            background-color: #fff; /* White background */
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0,0,0,0.1); /* Soft shadow */
        }
        h1 {
            text-align: center;
            margin-bottom: 20px;
            color: #007BFF; /* Blue color for header */
        }
        .form-group {
            margin-bottom: 15px;
        }
        label {
            display: block;
            margin-bottom: 5px;
            color: #555; /* Dark gray label text color */
        }
        input[type="number"],
        input[type="text"],
        select {
            width: 100%;
            padding: 8px;
            box-sizing: border-box;
            border: 1px solid #ccc; /* Light gray border */
            border-radius: 4px;
        }
        button {
            display: block;
            width: 100%;
            padding: 10px;
            background-color: #007BFF; /* Blue button background */
            color: white;
            border: none;
            cursor: pointer;
            margin-top: 10px;
            border-radius: 4px;
        }
        button:hover {
            background-color: #0056b3; /* Darker blue on hover */
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 20px;
        }
        th, td {
            border: 1px solid #ddd;
            padding: 8px;
            text-align: center;
        }
        th {
            background-color: #f4f4f4; /* Light gray header background */
        }
        .result {
            margin-top: 20px;
            padding: 10px;
            background-color: #e9f5f5; /* Light green background for result */
            border-radius: 4px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>KALAT Mo: Knapsack Allocation of Litterbins for Adaptive Trash Modulation</h1>
        <form id="allocationForm">
            <div class="form-group">
                <label for="numLocations">Number of Locations:</label>
                <input type="number" id="numLocations" required min="1">
            </div>
            <div class="form-group">
                <label for="budget">Total Budget for Waste Bins (₱):</label>
                <input type="number" id="budget" required min="1" step="0.01">
            </div>
            <div class="form-group">
                <label for="minBins">Minimum Number of Bins per Location:</label>
                <input type="number" id="minBins" required min="1">
            </div>
            <div class="form-group">
                <label for="maxBins">Maximum Number of Bins per Location:</label>
                <input type="number" id="maxBins" required min="1">
            </div>
            <div class="form-group">
                <label for="numBinTypes">Number of Bin Types:</label>
                <input type="number" id="numBinTypes" required min="1">
            </div>
            <div id="locationInputs"></div>
            <div id="binInputs"></div>
            <button type="button" onclick="generateInputs()">Generate Inputs</button>
            <button type="button" onclick="calculateAllocation()">Calculate Allocation</button>
        </form>
        <div id="result" class="result"></div>
        <div id="scoringGuide" style="display: none;">
            <h2>Scoring Guide</h2>
            <table>
                <thead>
                    <tr>
                        <th>Level</th>
                        <th>Foot Traffic (per day)</th>
                        <th>Waste Generation Rate (kg/day)</th>
                        <th>Environmental Impact (score out of 10)</th>
                    </tr>
                </thead>
                <tbody>
                    <tr>
                        <td>Low</td>
                        <td>100-500</td>
                        <td>10-50</td>
                        <td>1-3</td>
                    </tr>
                    <tr>
                        <td>Medium</td>
                        <td>501-1000</td>
                        <td>51-100</td>
                        <td>4-7</td>
                    </tr>
                    <tr>
                        <td>High</td>
                        <td>1001-2000</td>
                        <td>101-200</td>
                        <td>8-10</td>
                    </tr>
                </tbody>
            </table>
            <h2>Description of Levels</h2>
            <p><strong>Foot Traffic:</strong> Low: Suitable for locations with low population density, such as small parks or residential areas. Medium: Suitable for moderately busy locations, such as small shopping centers or medium-sized offices. High: Suitable for high-density areas, such as large malls, stadiums, or busy public transit stations.</p>
            <p><strong>Waste Generation Rate:</strong> Low: Represents locations with low waste production, possibly due to lower activity levels or effective waste reduction practices. Medium: Represents average waste production, typical of moderately busy areas. High: Represents high waste production, common in bustling areas or places with high consumption rates.</p>
            <p><strong>Environmental Impact:</strong> Low: Indicates minimal environmental impact, possibly due to effective waste management and low pollution. Medium: Indicates a moderate environmental impact, possibly due to moderate pollution or waste issues. High: Indicates a high environmental impact, possibly due to significant pollution or poor waste management practices.</p>
        </div>
        <button type="button" onclick="toggleDescription()">Show Description</button>
    </div>
    <script>
        // Define global variables to hold data
        let locations = [];
        let bins = [];

        // Function to swap two locations
        function swapLocations(a, b) {
            let temp = a;
            locations[a] = locations[b];
            locations[b] = temp;
        }

        // Function to swap two bin types
        function swapBins(a, b) {
            let temp = a;
            bins[a] = bins[b];
            bins[b] = temp;
        }

        // Function to partition locations based on value density
        function partitionLocations(low, high) {
            let pivot = locations[high].valueDensity;
            let i = low - 1;

            for (let j = low; j < high; j++) {
                if (locations[j].valueDensity > pivot) {
                    i++;
                    swapLocations(i, j);
                }
            }
            swapLocations(i + 1, high);
            return i + 1;
        }

        // Function to partition bins based on value ratio
        function partitionBins(low, high) {
            let pivot = bins[high].valueRatio;
            let i = low - 1;

            for (let j = low; j < high; j++) {
                if (bins[j].valueRatio > pivot) {
                    i++;
                    swapBins(i, j);
                }
            }
            swapBins(i + 1, high);
            return i + 1;
        }

        // Quicksort function for locations based on value density
        function quicksortLocations(low, high) {
            if (low < high) {
                let pi = partitionLocations(low, high);
                quicksortLocations(low, pi - 1);
                quicksortLocations(pi + 1, high);
            }
        }

        // Quicksort function for bins based on value ratio
        function quicksortBins(low, high) {
            if (low < high) {
                let pi = partitionBins(low, high);
                quicksortBins(low, pi - 1);
                quicksortBins(pi + 1, high);
            }
        }

        // Function to generate location input fields dynamically
        function generateLocationInputs(numLocations) {
            const container = document.getElementById('locationInputs');
            container.innerHTML = '';
            for (let i = 0; i < numLocations; i++) {
                const locationDiv = document.createElement('div');
                locationDiv.classList.add('form-group');
                locationDiv.innerHTML = `
                      <h3>Location ${i + 1}</h3>
                    <label for="locationName${i}">Location Name:</label>
                    <input type="text" id="locationName${i}" required>
                    <label for="footTraffic${i}">Foot Traffic (1-10):</label>
                    <input type="number" id="footTraffic${i}" required min="1" max="10">
                    <label for="wasteGeneration${i}">Waste Generation Rate (1-10):</label>
                    <input type="number" id="wasteGeneration${i}" required min="1" max="10">
                    <label for="environmentalImpact${i}">Environmental Impact (1-10):</label>
                    <input type="number" id="environmentalImpact${i}" required min="1" max="10">
                `;
                container.appendChild(locationDiv);
            }
        }

        // Function to generate bin input fields dynamically
        function generateBinInputs(numBinTypes) {
            const container = document.getElementById('binInputs');
            container.innerHTML = '';
            for (let i = 0; i < numBinTypes; i++) {
                const binDiv = document.createElement('div');
                binDiv.classList.add('form-group');
                binDiv.innerHTML = `
                    <h3>Bin Type ${i + 1}</h3>
                    <label for="binPrice${i}">Price (₱):</label>
                    <input type="number" id="binPrice${i}" step="0.01" required min="0.01">
                    <label for="binWeightCapacity${i}">Weight Capacity (kg):</label>
                    <input type="number" id="binWeightCapacity${i}" step="0.01" required min="0.01">
                    <label for="binValueRatio${i}" style="display:none;">Value Ratio:</label>
                    <input type="number" id="binValueRatio${i}" step="0.01" required style="display:none;" min="0.01">
                    <br><br>
                `;
                container.appendChild(binDiv);
            }
        }

        // Function to generate input fields based on user input
        function generateInputs() {
            const numLocations = parseInt(document.getElementById('numLocations').value);
            const numBinTypes = parseInt(document.getElementById('numBinTypes').value);
            
            generateLocationInputs(numLocations);
            generateBinInputs(numBinTypes);
        }

        // Function to calculate allocation based on user input
        function calculateAllocation() {
            const numLocations = parseInt(document.getElementById('numLocations').value);
            const budget = parseFloat(document.getElementById('budget').value);
            const minBins = parseInt(document.getElementById('minBins').value);
            const maxBins = parseInt(document.getElementById('maxBins').value);
            const numBinTypes = parseInt(document.getElementById('numBinTypes').value);

            // Clear previous results if any
            const resultDiv = document.getElementById('result');
            resultDiv.innerHTML = '';

            // If no inputs are provided, display a message and return
            if (isNaN(numLocations) || isNaN(budget) || isNaN(minBins) || isNaN(maxBins) || isNaN(numBinTypes)) {
                resultDiv.innerHTML = '<p>Please generate inputs and calculate again.</p>';
                return;
            }

            // Initialize locations array
            locations = [];
            for (let i = 0; i < numLocations; i++) {
                const name = document.getElementById(`locationName${i}`).value;
                const footTraffic = parseInt(document.getElementById(`footTraffic${i}`).value);
                const wasteGeneration = parseInt(document.getElementById(`wasteGeneration${i}`).value);
                const environmentalImpact = parseInt(document.getElementById(`environmentalImpact${i}`).value);
                const valueDensity = (footTraffic + wasteGeneration + environmentalImpact) / 3.0;

                locations.push({
                    name: name,
                    footTraffic: footTraffic,
                    wasteGeneration: wasteGeneration,
                    environmentalImpact: environmentalImpact,
                    valueDensity: valueDensity
                });
            }

            // Sort locations based on value density in descending order
            locations.sort((a, b) => b.valueDensity - a.valueDensity);

            // Initialize bins array
            bins = [];
            for (let i = 0; i < numBinTypes; i++) {
                const price = parseFloat(document.getElementById(`binPrice${i}`).value);
                const weightCapacity = parseFloat(document.getElementById(`binWeightCapacity${i}`).value);
                const valueRatio = weightCapacity / price;

                bins.push({
                    id: i + 1,
                    price: price,
                    weightCapacity: weightCapacity,
                    valueRatio: valueRatio
                });
            }

            // Sort bins based on value ratio in descending order
            bins.sort((a, b) => b.valueRatio - a.valueRatio);

            // Allocate bins to locations
            const allocations = [];
            let totalCost = 0;

            for (let i = 0; i < numLocations; i++) {
                allocations[i] = {};
                allocations[i].location = locations[i].name;
                allocations[i].totalBins = 0;
                allocations[i].bins = [];

                let remainingBins = minBins;
                for (let j = 0; j < numBinTypes && remainingBins > 0; j++) {
                    allocations[i].bins[j] = 0;
                    while (remainingBins > 0 && totalCost + bins[j].price <= budget && allocations[i].bins[j] < maxBins) {
                        allocations[i].bins[j]++;
                        totalCost += bins[j].price;
                        remainingBins--;
                        allocations[i].totalBins++;
                    }
                }

                let currentBins = minBins; // Start with minimum bins allocated
                for (let j = 0; j < numBinTypes; j++) {
                    while (totalCost + bins[j].price <= budget && currentBins < maxBins) {
                        if (!allocations[i].bins[j]) {
                            allocations[i].bins[j] = 0;
                        }
                        allocations[i].bins[j]++;
                        totalCost += bins[j].price;
                        currentBins++;
                        allocations[i].totalBins++;
                    }
                    if (totalCost >= budget) {
                        break;
                    }
                }
            }

            // Display results
            let resultHTML = '<h2>Result</h2>';
            resultHTML += `<p>Number of Locations: ${numLocations}</p>`;
            resultHTML += `<p>Budget: ₱${budget.toFixed(2)}</p>`; // Change dollar sign to peso sign here
            resultHTML += `<p>Minimum Bins per Location: ${minBins}</p>`;
            resultHTML += `<p>Maximum Bins per Location: ${maxBins}</p>`;
            resultHTML += `<p>Locations:</p>`;
            resultHTML += '<ul>';
            allocations.forEach(allocation => {
                resultHTML += `<li>Location: ${allocation.location} - Total Bins: ${allocation.totalBins}</li>`;
                resultHTML += '<ul>';
                allocation.bins.forEach((binsAllocated, index) => {
                    resultHTML += `<li>Bin Type ${bins[index].id}: ${binsAllocated}</li>`;
                });
                resultHTML += '</ul>';
            });
            resultHTML += '</ul>';
            resultHTML += `<p>OVERALL:</p>`;
            resultHTML += `<p>Total Budget Allocated: ₱${totalCost.toFixed(2)}</p>`; // Change dollar sign to peso sign here
            resultHTML += `<p>Remaining Budget: ₱${(budget - totalCost).toFixed(2)}</p>`; // Change dollar sign to peso sign here

            resultDiv.innerHTML = resultHTML;
        }

        // Function to toggle scoring guide visibility
        function toggleDescription() {
            const scoringGuide = document.getElementById('scoringGuide');
            scoringGuide.style.display = scoringGuide.style.display === 'none' ? 'block' : 'none';
        }

        // Initialize with default inputs
        generateInputs();
    </script>
</body>
</html>
