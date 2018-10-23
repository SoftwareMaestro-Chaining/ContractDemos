pragma solidity ^0.4.24;

// authority
contract auth {
    address public owner;

    function auth() {
        owner = msg.sender;
    }

    modifier onlyOwner {
        if (msg.sender != owner) revert();
        _;
    }

    function kill() onlyOwner {
        selfdestruct(owner);
    }
}

contract Advertiser is auth {

    mapping(address => Advertise[]) public advertises;
    mapping(uint => User[]) public addressRegisterMember;
    
    struct Advertise {
        uint index; // pk
        string url;
        string name;
        uint length;
        uint minViews;
        uint funds;
    }
    
    struct User {
        address userAddress;
        uint views;
    }
    
    // add advertise
    function addAdvertise(string _url, string _name, uint _length, uint _minViews, uint _funds) onlyOwner{
        if(msg.sender != owner) revert();
        
        uint _index = advertises[msg.sender].length + 1;
        
        advertises[msg.sender].push(Advertise({
            index: _index,
            url : _url,
            name : _name,
            length : _length,
            minViews : _minViews,
            funds : _funds
        }));
    }
    
    function isMember (uint advertiseIndex) returns(bool){
        uint length = addressRegisterMember[advertiseIndex].length;
        for(uint i=0; i<length; i++) {
            if(addressRegisterMember[advertiseIndex][i].userAddress == msg.sender) {
                return true;
            }
        }
        return false;
    }

    // get user pk    
    function getUserIndex(address user, uint advertiseIndex) view returns(uint) {
        uint length = addressRegisterMember[advertiseIndex].length;
        for(uint i=0; i<length; i++) {
            if(addressRegisterMember[advertiseIndex][i].userAddress == user) {
                return i;
            }
        }
        revert();
    }
    
    // after earned views
    function checkViews(uint advertiseIndex, uint views) view returns(uint){
        require(isMember(advertiseIndex));
        uint memberIndex = getUserIndex(msg.sender, advertiseIndex);
        uint accumulateViews = addressRegisterMember[advertiseIndex][memberIndex].views;
        return views - accumulateViews;
    }
    
    // ad rewards
    function refunds(uint advertiseIndex, uint views) {
        require(isRefundable(advertiseIndex, views));
        require(isMember(advertiseIndex));
        uint currentViews = checkViews(advertiseIndex, views);
        addViews(advertiseIndex, currentViews);
        uint refundValue = currentViews / advertises[owner][advertiseIndex].minViews;
        refundValue = refundValue * 500000000000000; // 1$ per 1000 views
        msg.sender.send(refundValue);
    }
    
    function addViews(uint advertiseIndex, uint views) {
        require(isMember(advertiseIndex));
        uint memberIndex = getUserIndex(msg.sender, advertiseIndex);
        addressRegisterMember[advertiseIndex][memberIndex].views += views;
    }
    
    function isRefundable (uint advertiseIndex, uint views) view returns(bool){
        if(checkViews(advertiseIndex, views) >= advertises[owner][advertiseIndex].minViews) {
            return true;
        } else {
            return false;
        }
    }
    
    function addBalance() payable onlyOwner {
        if(msg.sender != owner) revert();
    }
    
    // register user address in advertise
    function registerMember(uint advertiseIndex) public {
        addressRegisterMember[advertiseIndex].push(User({
            userAddress: msg.sender,
            views: 0
        }));
    }
    
    function checkAdvertiseBalance() public view returns(uint){
        return address(this).balance;
    }
}