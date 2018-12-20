=== Pseudo code art auction ===

// Notice: Normally, a winner needs to claim his win in a smart contract based auctions, becaus claiming the 
// good a bidder won cannot be triggered automatically just because currentBlock equals or is greater than auctionEndBlock ;)
// So a transaction is needed. In this case, I suggest doing the following:
// If currentBlock >= auctionEndBlock, let's display this auction as finished in UI and tell the user 
// that a new auction has started for this slot. The bidding on the slot will then, in the smart contract (!),
// actually mark the previous auction as finished and start a new one. If no bids come in for newly created auctions, 
// either admin functions could trigger the beforementioned thing ( so the drone can start painting the winner's image finally)
// or crew members simply place bids on the auction themselves.

// I propose for the smart contract to not keep track of which painting was already dealt with, else the whole project 
// gets more complicated and buggy, this should be handled elsewhere.

// For none of the functions I checked if the required parameters were actually set by the user's transaction,
// is this checked automatically on some level actually? 

// So here we go:

// define admins, maybe with a mapping?
address => bool : map isAdmin

// have a unique ID for every auction, start with zero and count up by one. this will also serve as an index later
// for listing / handling all auctions.
uniqueAuctionId: int;

// save all past and ongoing auctions in a map (or the sofia equivalent of an array). Because the uniqueAuctionId is a counter
// it will be easy to iterate over this data later to retrieve every entry as we know all the key values for this map.
// "auction" type is defined a little further below.
int => auction : map allAuctions;

// have a map that to keep track of the latest auction of each slot, map a slot (as input) to a unique auction ID (as output)
// whenever a new auction starts in a slot, you overwrite the value for the latest auction of a slot
int => int : map latestAuctionBySlotId

// define a bid;
bid = {
    bidder: address // bidder, obviously
    bid: int // bid amount
    pictureID: *something* // (you're going to put the unique artwork identifier here later, so pick some suitable data type)
}

// define an auction: 
{
    id: int,  (the unique auction ID)
    slot: int, (for easier handling/displaying in UI later, when reading contract data)
    highestBid: bid (previously defined bid type)
    active: bool (is the auction still active or not? :) )
    endBlock: int (here comes the blocknumber of the block when the auction is supposed to end)

    // I know "active" could possibly be derived from checking if (currentblock>=endBlock), but let's save on code execution here?
}

// optional: Have a map that draws a relation between auctions and the unique identifier of the winning bidder's picture
// this information could be gathered elsehow, too, but transactions are cheap still so we make our life easier
// again, you can use an array-equivalent here, too. The key values will always be 0,1,2,3,4...

int => *something* : map winningPictureIdByAuctionID

// have a function to start a new auction. 

// CAUTION: make this function be triggerable ONLY internal or by an admin. 
// if not possible, put two implementations of this func into the smart contract, one for admins only and one for private only.
// This is called in a private way by the placeBid() func !

startAuction (_slot: int, (optional: _blockDuration: int, *someOtherParamsEventually*)  ) stateful : int =

    // make sure this can only be called from within this contract or by admins
    require(Call.caller == this || isAdmin[Call.caller])

    // check if the last auction of the _slot has reached its endblock yet.
    // Following line says: "get the auction ID of the latest auction of that _slot and ask for that auction's endblock, then check if endblock is reached already"
    if(allAuctions[latestAuctionBySlotId[_slot]].endBlock >= theCurrentBlock){

        // endblock is reached! Do four things: 
        
        // define a helper: the ID of the last auction of the _slot
        let endedAuctionID : int = latestAuctionBySlotId[_slot].id;

        // 1. set the last auction of this _slot to finished 
        allAuctions[endedAuctionID].active = false;

        //2. put the picture identifier of the winning bid in the winningPictureIdByAuctionID map
        winningPictureIdByAuctionID[endedAuctionID] = allAuctions[endedAuctionID].highestBid.pictureID
       
        
        //3. and create a new auction for the _slot
        
        //3.1 create the auction itself...
        let auction newAuction = {
            id: uniqueAuctionId 
            slot: _slot
            highestBid: null // or, if sophia doesn't like null, initiate a bid type val here with bogus zero-like values for the fields.
            active: true
            endBlock: (either do currentBlock + someValue or allow this to be set manually or whatever)

        // 3.2 ... put it in the allAuctions mapping...
        allAuctions[uniqueAuctionId] = newAuction // of course you can combine step 3.1 and 3.2

        // 3.3 ... set this new auction to be the new latest auction of this _slot
        latestAuctionBySlotId[_slot] = uniqueAuctionId;

        // 3.4 ...and don't forget to count up the uniqueAuctionId for the next use ;)
        uniqueAuctionId++;
        
        // 4. last but not least, withdraw the bidder's bid amount from the contract to somewhere !
        send(allAuctions[endedAuctionID].highestBid.bid).to(someWallet)
        
        // and return the uniqueAuctionId of the new auction, important for placeBid() !
        return uniqueAuctionId
        }
    } else {
        // endblock is not reached yet, abort operation.
        abort("Auction for the slot" + _slot + "still running" !) // have fun concatenating in sophia ;)
    }
}

// the function for bidding 
function placeBid (int : _slot, *uniquePictureIDdataType* : _picture) stateful payable {

    // define a helper first: the unique ID of the auction we're talking about here 
    var thisAuctionId : int = allAuctions[latestAuctionBySlotId[_slot]].id

    // then, check if the latest auction of that slot has maybe reached its endblock already
    if (allAuctions[thisAuctionId].endBlock >= currentBlock){
        // if that's the case, create a new auction for that _slot and set thisAuctionId to the ID
        // of the new auction we create here (it's returned by startAuction() )
        // Not sure if sophia's scoping rules will let you do that easily, got my fingers crossed for you guys

        thisAuctionId = startAuction(_slot);
    }
    // check if tis bid's value is higher than the last one for the _slot 's current auction
    // optional: you can require the bid to be at least X amount higher than the latest one 
    require (call.value + X > allAuctions[thisAuctionId].highestBid.bid)

    // so if this bid is higher than the last one indeed, we set the current bid/bidder as the new leading bid
    allAuctions[thisAuctionId].highestBid = {
        bid: call.value,
        bidder: call.caller,
        pictureID: _picture
    }
}

// helper function for retrieving all finished auctions' picture IDs for one slot. Note that this is run locally so we 
// don't care about escessive looping / if condition checking.
// of course, this could also be performed through calls to allAuctions from without the contract ( see getAuctionById() ), but i'd prefer this.

// Caution: If there are limitations on return data size, perform the getting of finished auctions off-contract with the getter function getAuctionByIdI() !
getFinishedAuctionsPictureIDsBySlot(_slot : int) : *data type of unique picture identifier* {
    
    // define the return data 
    let allWinningPicturesBySlot[];
    
    // iterate through all auctions...
    for (i=0; i<uniqueAuctionId, i++){

        // ... and if one auction's ID matches the slot we look for...
        if ( allAuctions[i].slot == _slot) {

            // ...get the winning bid's picture ID and push it to the return data
            allWinningPictures.push(allAuctions[i].highestBid.pictureID)
        }
    }

    // return the IDs of all pictures that won an auction across all slots
    return allWinningPicturesBySlot;
}


// get one auction byID, also run locally
//hopefully it's possible to return an own-defined, struct-like datatype as a whole, if not you'll have to help yourselves.. ;)
function getAuctionById(int : _auction) : auction {
    return allAuctions[_auction];
}

// if you have read until here, notice the following: If an auction is started and nobody bids for 
// it until the endblock, the ending of that auction will make it simply have no winner, and that's okay. 
// the new creation of a bid creates an auction that will, from that moment, run for the predefined amount of blocks.
