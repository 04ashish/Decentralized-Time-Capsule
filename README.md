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
import React, { useState, useEffect } from 'react';
import Web3 from 'web3';
import TimeCapsuleABI from './TimeCapsuleABI.json';

const App: React.FC = () => {
  const [web3, setWeb3] = useState<Web3 | null>(null);
  const [contract, setContract] = useState<any>(null);
  const [account, setAccount] = useState<string>('');
  const [message, setMessage] = useState<string>('');
  const [revealDate, setRevealDate] = useState<string>('');

  useEffect(() => {
    const initWeb3 = async () => {
      if (window.ethereum) {
        const web3Instance = new Web3(window.ethereum);
        try {
          await window.ethereum.enable();
          setWeb3(web3Instance);
          const accounts = await web3Instance.eth.getAccounts();
          setAccount(accounts[0]);
          const contractAddress = '0x...'; // Replace with your deployed contract address
          const contractInstance = new web3Instance.eth.Contract(TimeCapsuleABI, contractAddress);
          setContract(contractInstance);
        } catch (error) {
          console.error("User denied account access");
        }
      }
    };
    initWeb3();
  }, []);

  const storeMessage = async () => {
    if (!web3 || !contract) return;
    const encryptedMessage = web3.utils.asciiToHex(message);
    const revealTimestamp = Math.floor(new Date(revealDate).getTime() / 1000);
    await contract.methods.storeMessage(encryptedMessage, revealTimestamp).send({ from: account });
    setMessage('');
    setRevealDate('');
  };

  return (
    <div>
      <h1>Decentralized Time Capsule</h1>
      <input
        type="text"
        value={message}
        onChange={(e) => setMessage(e.target.value)}
        placeholder="Enter your message"
      />
      <input
        type="datetime-local"
        value={revealDate}
        onChange={(e) => setRevealDate(e.target.value)}
      />
      <button onClick={storeMessage}>Store Message</button>
    </div>
  );
};

export default App;

