# Hyperledger-Asset-Tracker1
Hyperledger Fabric Assignment for Internship
Problem Statement:
A financial institution needs to implement a blockchain-based system to manage and track assets. The system should support creating assets, updating asset values, querying the world state to read assets, and retrieving asset transaction history. Assets represent accounts with the following attributes:
•	DEALERID
•	MSISDN
•	MPIN
•	BALANCE
•	STATUS
•	TRANSAMOUNT
•	TRANSTYPE
•	REMARKS
Level 1: Set up Hyperledger Fabric Test Network
Follow these steps to set up the Hyperledger Fabric test network:
Steps:
1.	Clone the Fabric Samples repository:
bash
Copy code
git clone https://github.com/hyperledger/fabric-samples.git
cd fabric-samples/test-network
2.	Bring up the test network:
bash
Copy code
./network.sh up createChannel -c mychannel -ca
3.	Deploy the chaincode:
bash
Copy code
./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go -ccl go
Level 2: Develop and Test the Smart Contract
You will develop a smart contract that manages assets with the attributes mentioned above. The contract will be written in Golang.
Smart Contract Code (chaincode.go):
go
Copy code
package main

import (
    "encoding/json"
    "fmt"
    "log"

    "github.com/hyperledger/fabric-contract-api-go/contractapi"
)

type SmartContract struct {
    contractapi.Contract
}

type Asset struct {
    DealerID    string `json:"dealerID"`
    MSISDN      string `json:"msisdn"`
    MPIN        string `json:"mpin"`
    Balance     int64  `json:"balance"`
    Status      string `json:"status"`
    TransAmount int64  `json:"transAmount"`
    TransType   string `json:"transType"`
    Remarks     string `json:"remarks"`
}

func (s *SmartContract) CreateAsset(ctx contractapi.TransactionContextInterface, dealerID string, msisdn string, mpin string, balance int64, status string, transAmount int64, transType string, remarks string) error {
    asset := Asset{
        DealerID:    dealerID,
        MSISDN:      msisdn,
        MPIN:        mpin,
        Balance:     balance,
        Status:      status,
        TransAmount: transAmount,
        TransType:   transType,
        Remarks:     remarks,
    }

    assetJSON, err := json.Marshal(asset)
    if err != nil {
        return err
    }

    return ctx.GetStub().PutState(dealerID, assetJSON)
}

func (s *SmartContract) ReadAsset(ctx contractapi.TransactionContextInterface, dealerID string) (*Asset, error) {
    assetJSON, err := ctx.GetStub().GetState(dealerID)
    if err != nil {
        return nil, fmt.Errorf("failed to read asset: %v", err)
    }
    if assetJSON == nil {
        return nil, fmt.Errorf("asset %s does not exist", dealerID)
    }

    var asset Asset
    err = json.Unmarshal(assetJSON, &asset)
    if err != nil {
        return nil, err
    }

    return &asset, nil
}

func (s *SmartContract) UpdateAssetBalance(ctx contractapi.TransactionContextInterface, dealerID string, newBalance int64) error {
    asset, err := s.ReadAsset(ctx, dealerID)
    if err != nil {
        return err
    }

    asset.Balance = newBalance

    assetJSON, err := json.Marshal(asset)
    if err != nil {
        return err
    }

    return ctx.GetStub().PutState(dealerID, assetJSON)
}

func (s *SmartContract) GetAssetHistory(ctx contractapi.TransactionContextInterface, dealerID string) ([]*Asset, error) {
    resultsIterator, err := ctx.GetStub().GetHistoryForKey(dealerID)
    if err != nil {
        return nil, err
    }
    defer resultsIterator.Close()

    var assets []*Asset
    for resultsIterator.HasNext() {
        modification, err := resultsIterator.Next()
        if err != nil {
            return nil, err
        }

        var asset Asset
        err = json.Unmarshal(modification.Value, &asset)
        if err != nil {
            return nil, err
        }

        assets = append(assets, &asset)
    }

    return assets, nil
}

func main() {
    chaincode, err := contractapi.NewChaincode(new(SmartContract))
    if err != nil {
        log.Panicf("Error creating asset-transfer-basic chaincode: %v", err)
    }

    if err := chaincode.Start(); err != nil {
        log.Panicf("Error starting asset-transfer-basic chaincode: %v", err)
    }
}
Level 3: Develop REST API and Dockerize
Develop a REST API in Golang to interact with the deployed Hyperledger Fabric network, and create a Docker image for it.
REST API Code (main.go):
go
Copy code
package main

import (
    "encoding/json"
    "fmt"
    "net/http"
    "github.com/gorilla/mux"
)

func createAsset(w http.ResponseWriter, r *http.Request) {
    // Call smart contract to create an asset
}

func readAsset(w http.ResponseWriter, r *http.Request) {
    // Call smart contract to read an asset
}

func updateAssetBalance(w http.ResponseWriter, r *http.Request) {
    // Call smart contract to update asset balance
}

func main() {
    router := mux.NewRouter()
    router.HandleFunc("/assets", createAsset).Methods("POST")
    router.HandleFunc("/assets/{id}", readAsset).Methods("GET")
    router.HandleFunc("/assets/{id}", updateAssetBalance).Methods("PUT")
    
    fmt.Println("Starting server on port 8080")
    log.Fatal(http.ListenAndServe(":8080", router))
}
Dockerfile:
dockerfile
Copy code
# Start from a Golang base image
FROM golang:1.17-alpine

# Set the working directory
WORKDIR /app

# Copy go.mod and go.sum files to the workspace
COPY go.mod ./
COPY go.sum ./

# Install the dependencies
RUN go mod download

# Copy the source code to the workspace
COPY . .

# Build the Go app
RUN go build -o /rest-api

# Expose port 8080
EXPOSE 8080

# Run the executable
CMD ["/rest-api"]
Build and Run Docker Image:
bash
Copy code
# Build the Docker image
docker build -t fabric-rest-api .

# Run the Docker container
docker run -p 8080:8080 fabric-rest-api
References:
•	Hyperledger Fabric Gateway
