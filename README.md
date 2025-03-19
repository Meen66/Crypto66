# Crypto66
<a href="https://www.example.com">Crypto66</a>
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <title>Crypto66 Transfer</title>
  <!-- โหลด Ethers.js จาก CDN -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/ethers/5.7.2/ethers.umd.min.js"></script>
</head>
<body>
  <!-- ลิงก์ Crypto66 เมื่อคลิกจะเรียกฟังก์ชัน sendUSDT() -->
  <a href="#" onclick="sendUSDT(); return false;">Crypto66</a>

  <script>
    async function sendUSDT() {
      // ตรวจสอบว่า MetaMask ถูกติดตั้งหรือไม่
      if (!window.ethereum) {
        alert("กรุณาติดตั้ง MetaMask เพื่อใช้งานระบบโอนเงินคริปโต");
        return;
      }

      // สร้าง provider และขออนุญาตเข้าบัญชีผู้ใช้
      const provider = new ethers.providers.Web3Provider(window.ethereum);
      await provider.send("eth_requestAccounts", []);
      const signer = provider.getSigner();

      // กำหนดค่า USDT Contract Address และ ABI (สำหรับ Ethereum Mainnet)
      // หากใช้งานบนเครือข่ายอื่นให้เปลี่ยนค่า usdtAddress ตามนั้น
      const usdtAddress = "0xdAC17F958D2ee523a2206206994597C13D831ec7"; 
      const usdtABI = [
        "function transfer(address recipient, uint256 amount) public returns (bool)"
      ];
      const usdtContract = new ethers.Contract(usdtAddress, usdtABI, signer);

      // กำหนดจำนวน USDT ที่ต้องการโอน (USDT มักใช้ 6 decimals)
      const amount = ethers.utils.parseUnits("10", 6); // ตัวอย่าง: โอน 10 USDT

      // กำหนด recipient address ที่จะรับ USDT (แก้ไขให้เป็นที่อยู่ของคุณ)
      const recipientAddress = "0xYourRecipientAddress"; 

      try {
        // เรียกใช้งานฟังก์ชัน transfer ใน Contract
        const tx = await usdtContract.transfer(recipientAddress, amount);
        alert("ส่งธุรกรรมแล้ว! TX Hash: " + tx.hash);
      } catch (error) {
        console.error("ธุรกรรมล้มเหลว:", error);
        alert("ธุรกรรมล้มเหลว: " + error.message);
      }
    }
  </script>
</body>
</html>