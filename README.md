(() => {
  const norm = (s) => (s || "").replace(/\s+/g, " ").trim();
  const sleep = (ms) => new Promise((r) => setTimeout(r, ms));

  /* ========================================================
      CONFIG: 10 VÒNG - TẬP TRUNG 3 DIGIT (TĂNG TỐC 10%)
     ======================================================== */
  const MY_PASS = "your account password VD: 0,0,0,0,0,0";
  const AMOUNT_VALUE = "99"; 
  const ROUNDS_DATA = [
    ["001", "000", "000", "000", "000"], ["101", "111", "111", "111", "111"],
    ["201", "222", "222", "222", "222"], ["301", "333", "333", "333", "333"],
    ["401", "444", "444", "444", "444"], ["501", "555", "555", "555", "555"],
    ["601", "666", "666", "666", "666"], ["701", "777", "777", "777", "777"],
    ["801", "888", "888", "888", "888"], ["901", "999", "999", "999", "999"],["001", "900", "100", "199", "299"],
  ];

  let currentSlotElement = null; 
  let currentRoundIndex = 0; 
  let isRunning = false; 
  let isWaitingNextSlot = false;
  
  let plusClicked = 0; 
  let digitFilled = false; 
  let buyNowClicked = false; 
  let confirmClicked = false;

  const setNativeValue = (input, value) => {
    if (!input) return;
    const setter = Object.getOwnPropertyDescriptor(window.HTMLInputElement.prototype, "value")?.set;
    if (setter) setter.call(input, value);
    else input.value = value;
    input.dispatchEvent(new Event("input", { bubbles: true }));
  };

  const fireClick = (el) => {
    if (!el) return false;
    const opts = { bubbles: true, cancelable: true, view: window };
    el.dispatchEvent(new MouseEvent("mousedown", opts));
    el.dispatchEvent(new MouseEvent("mouseup", opts));
    el.click();
    return true;
  };

  const fillRow = (row, value) => {
    const inputs = Array.from(row.querySelectorAll("input"));
    const amountInp = row.querySelector("input.el-input__inner");
    if (amountInp) setNativeValue(amountInp, AMOUNT_VALUE);

    const digitInps = inputs.filter(inp => inp !== amountInp);
    if (digitInps.length === 1) {
      setNativeValue(digitInps[0], value);
    } else if (digitInps.length > 1) {
      const chars = value.split("");
      chars.forEach((char, idx) => {
        if (digitInps[idx]) setNativeValue(digitInps[idx], char);
      });
    }
  };

  const main = async () => {
    if (isRunning) return; 
    isRunning = true;

    const liveWrapper = [...document.querySelectorAll(".item-wrapper")]
      .find(w => norm(w.querySelector(".live-label")?.textContent) === "Live");

    if (liveWrapper) {
      const liveBtn = liveWrapper.querySelector(".item-btn");
      if (liveBtn && liveBtn !== currentSlotElement) {
        currentSlotElement = liveBtn;
        currentRoundIndex = 0; 
        isWaitingNextSlot = false;
        plusClicked = 0; digitFilled = false; buyNowClicked = false; confirmClicked = false;
        fireClick(liveBtn); 
        await sleep(400); // Giảm từ 500
      }
    }

    if (isWaitingNextSlot) { isRunning = false; return; }

    const closeBtn = document.querySelector(".close-btn, .van-dialog__confirm, .van-button--default");
    if (closeBtn && closeBtn.offsetParent !== null) {
      await sleep(320); // Giảm từ 400
      if (fireClick(closeBtn)) {
        if (currentRoundIndex >= ROUNDS_DATA.length - 1) {
          isWaitingNextSlot = true;
        } else {
          currentRoundIndex++;
          plusClicked = 0; digitFilled = false; buyNowClicked = false; confirmClicked = false;
        }
      }
    }

    const isModeActive = [...document.querySelectorAll(".selected, .active")]
      .some(el => el.textContent.includes("3 Digit"));

    if (!isModeActive) {
      const modeBtn = [...document.querySelectorAll("button, div")]
        .find(el => norm(el.textContent) === "3 Digit");
      if (modeBtn) {
        fireClick(modeBtn);
        await sleep(800); // Giảm từ 1000
      }
    }

    const currentRows = document.querySelectorAll(".buy-item").length;
    if (currentRows < 5) {
      const plusBtn = document.querySelector(".add-item") || 
                     [...document.querySelectorAll("span, div")].find(e => e.textContent.trim() === "+" && e.offsetWidth > 0);
      if (plusBtn) {
        fireClick(plusBtn);
        await sleep(240); // Giảm từ 300
      }
    }

    const rows = document.querySelectorAll(".buy-item");
    if (!digitFilled && rows.length >= 5) {
      ROUNDS_DATA[currentRoundIndex].forEach((val, i) => fillRow(rows[i], val));
      digitFilled = true;
    }

    if (digitFilled && !buyNowClicked) {
      const buyBtn = document.querySelector(".buynow_btn") || 
                     [...document.querySelectorAll("button, div")].find(el => norm(el.textContent).toUpperCase() === "BUY NOW");
      if (fireClick(buyBtn)) {
        buyNowClicked = true;
        await sleep(480); // Giảm từ 600
      }
    }

    if (buyNowClicked && !confirmClicked) {
      const passInps = document.querySelectorAll('input[type="password"], .van-password-input__security li');
      const hiddenPassInp = document.querySelector('input[pattern="[0-9]*"], input[maxlength="6"]');
      
      if (hiddenPassInp) {
        setNativeValue(hiddenPassInp, MY_PASS);
      } else if (passInps.length >= 6) {
        for (let i = 0; i < 6; i++) {
          setNativeValue(passInps[i], MY_PASS[i]);
        }
      }

      await sleep(560); // Giảm từ 700
      const okBtn = [...document.querySelectorAll("button, .van-button--primary")]
        .find(el => /CONFIRM|XÁC NHẬN|OK/i.test(norm(el.textContent)));
      if (okBtn && fireClick(okBtn)) {
        confirmClicked = true;
      }
    }

    isRunning = false;
  };

  setInterval(main, 720); // Nhịp quét nhanh hơn 20% (900 * 0.8)
  console.log("🚀 BOT TĂNG TỐC 10% ĐÃ KÍCH HOẠT.");
})();
