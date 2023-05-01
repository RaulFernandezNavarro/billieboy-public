# BILLIEBOY

## What is this?

The Billieboy project is a challenge for IT students attending [bol.com](http://bol.com/)'s in-house day. It's designed to teach students about microservices architectures, which are increasingly used by large companies like [bol.com](http://bol.com/) to develop scalable and resilient systems. Through the project, students will learn how to interact with microservices and gain practical experience working with these systems. It's a great opportunity to prepare for the real-world challenges of working in the tech industry.

## Installation

### What do you need?

- Ready-to-use IDE of preference. Jupyter notebooks are acceptable if hosted locally, but **not in** **Google Colab**.
- Docker installed ([https://docs.docker.com/get-docker/](https://docs.docker.com/get-docker/))

### Download docker images

After installing Docker, you'll need to download two images: one containing various microservices, and the other containing a front-end.

******************Microservices:******************

```docker
docker pull raulfernandeznavarro/billieboy-image:1.0.0
```

********************Front-end:********************

```docker
docker pull raulfernandeznavarro/billieboy-frontend:1.0.0
```

### Run docker images

Once you have obtained the images, you should run them. In two different terminal windows, run the following commands and do not close them until the challenge is complete.

******************Microservices:******************

```docker
docker run -p 8000:8000 raulfernandeznavarro/billieboy-image:1.0.0
```

********************Front-end:********************

```docker
docker run -p 8500:8500 raulfernandeznavarro/billieboy-frontend:1.0.0
```

After running them, you should be able to access the microservices’ documentation here → [http://localhost:8000/docs#/](http://localhost:8000/docs#/) . The UI is not yet connected to any service, so it will display some messages that you can ignore for now. You should be able to access the UI here → [http://localhost:8500/](http://localhost:8500/) .

## Challenge

Have you ever wondered what happens once you return a package? Let’s see!

Once you hand your parcel to the transport company, the return package is sent to the **[bol.com](http://bol.com/)** return warehouse. There, it is assessed by expert operators to determine its condition, and a rule engine decides where the package should go. For example, items that were never opened can be resold, while items that are broken beyond repair must be destroyed. But what happens when an item is not good enough to be resold but also not broken enough to be destroyed? In that case, it is sold to wholesalers for a fraction of the original price. Until now, different wholesalers had agreements with bol.com about what types of items they buy. However, due to increasing demand and to make the market more competitive, we are creating a bidding platform for wholesalers. This platform will allow them to place bids for the items they want to purchase.

Your job will be to create the bidding platform microservice, which will communicate with other backend services to obtain the necessary information and forward it to a UI for customers to bid on.

![Screenshot 2023-04-30 at 22.56.17.png](./img/Screenshot%202023-04-30%20at%2022.56.17.png)

## The landscape - Microservices

Remember: 
A GET endpoint is an HTTP method used to request data from a server

A POST endpoint is used to submit some data to be processed by a server. Sometimes, POST can also be used as a GET (to retrieve some information) when you need to pass along some information in the body of the request. 

Glosary:

- EAN (European Article Number): is a code used to identify products in retail settings. An EAN is associated with a single product (e.g., iPhone 13 Pro)
- Defect_id: Is a unique identifier for a particular item that is defect (e.g., the iPhone 13 Pro that you bought and gave back because it was scratched on the left corner)
- Grading: the process of evaluating the condition of a returned item (i.e., Is it good enough to be sold? Should it be destroyed? …)

![Screenshot 2023-04-30 at 23.37.30.png](./img/Screenshot%202023-04-30%20at%2023.37.30.png)

### Warehouse

This miroservice takes care of all the operations that are carried out in the warehouse. In here you can obtain an item, an item batch or a defect batch. You are also able to post a destination for a given item.

### Catalog

This service holds the product catalog. You can request the information for any item by posting here its EAN.

### Operator

This microservice is connected to the machines that the operators use to grade items. You can post here a defectId + EAN to obtain the grading of a specific defect.

### Finance

This service takes care of most of the money related transactions. In the get-resell-value endpoint, you can post a defect and a grade to obtain the value for which the item can be resold. In the invoicing endpoint, you can post a financial transaction, and the system will take care of executing it.

## FIRST CHALLENGE - Display items in the UI

For this first challenge, you will create a very easy web application with only one endpoint. You will obtain an item batch from the warehouse API and serve it through a GET endpoint to the UI. The UI will make a GET request to [localhost:8002/ui/item-batch/](http://localhost:8002/ui/item-batch/) so make sure to spin up your endpoint there. These are the steps you should follow, but feel free to do them in a different order:

- Make a request, from the code, to the item-batch endpoint.
- Create a simple web application (there are many frameworks for this: FastAPI for python, Springboot for java/kotlin, …)
- Create a get endpoint in your application. The logic inside the endpoint should be: get item batch → serve item batch

That’s it! Of course, this is a simplified version, but good job!

## SECOND CHALLENGE - Obtain the information hard mode

Although cool, in real life the previous scenario is hardly ever the case. It wouldn’t make a lot of sense to just
forward some information. This second challenge will show more in depth how microservice architectures really work. The
main goal is to collaborate between microservices to provide certain functionality. In our case, we provide a bidding
platform, but will have to collaborate with all other 4 services to make this happen.

In this challenge, you will construct the item batch to feed to the UI by merging pieces that you will get from the
different services. The following diagram shows a high-level representation of the steps that you should take. Check the
APIs documentation to see where you can find each information.

![Screenshot 2023-05-01 at 00.03.25.png](./img/Screenshot%202023-05-01%20at%2000.03.25.png)

## EXTRA CHALLENGE - What comes after
Up to this point, we have taken care of half of the work. We have obtained information from other services to feed our
UI and it looks great! But it doesn't really do anything yet. Currently, users can see items and make bids in the
frontend. Now, your service needs to do something with those bids.

The UI is designed in the following manner. It displays an item and its bids, allowing you to also bid on items. Once you hit the refresh item button, the front-end will do the follwing:

- Send a POST request with all the bids to your service. More specifically, the UI expects to POST this information to [localhost:8002/ui/item-bids/](http://localhost:8002/ui/item-bids/), so make sure that your service is running on port 8002 and that there is a POST endpoint in that address.
- Display a new item.

![Screenshot 2023-05-01 at 00.05.14.png](./img/Screenshot%202023-05-01%20at%2000.05.14.png)

some useful definitions:

```python
class MonetaryValue(BaseModel):
    def __init__(self, currency: str, value: float):
        super().__init__(currency=currency, value=value)
        self.currency = currency
        self.value = value

    currency: str
    value: float


class Bid(BaseModel):
    def __init__(self, bidding_party: str, bid_amount: MonetaryValue):
        super().__init__(bidding_party=bidding_party, bid_amount=bid_amount)
        self.bidding_party = bidding_party
        self.bid_amount = bid_amount

    bidding_party: str
    bid_amount: MonetaryValue


class Bids(BaseModel):
    def __init__(self, unique_id: str, bids: list[Bid]):
        super().__init__(unique_id=unique_id, bids=bids)
        self.unique_id = unique_id
        self.bids = bids

    unique_id: str
    bids: list[Bid]

```
