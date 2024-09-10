# Decentralized-Time-Capsule
A time capsule app where users can submit messages to be revealed at a future date. The blockchain will lock the messages and prevent early access, with the reveal date coded into the smart contract
pragma solidity ^0.8.0;

contract TimeCapsule {
    struct Message {
        address sender;
        string encryptedContent;
        uint256 revealDate;
        bool revealed;
    }

    Message[] public messages;

    event MessageStored(uint256 indexed messageId, address sender, uint256 revealDate);
    event MessageRevealed(uint256 indexed messageId, string content);

    function storeMessage(string memory _encryptedContent, uint256 _revealDate) public {
        require(_revealDate > block.timestamp, "Reveal date must be in the future");
        messages.push(Message(msg.sender, _encryptedContent, _revealDate, false));
        emit MessageStored(messages.length - 1, msg.sender, _revealDate);
    }

    function revealMessage(uint256 _messageId) public {
        Message storage message = messages[_messageId];
        require(block.timestamp >= message.revealDate, "Message cannot be revealed yet");
        require(!message.revealed, "Message has already been revealed");

        message.revealed = true;
        emit MessageRevealed(_messageId, message.encryptedContent);
    }

    function getMessageCount() public view returns (uint256) {
        return messages.length;
    }
}
