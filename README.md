/**
 * 🔒 ตั้งค่าความปลอดภัยระบบ 
 */
var ADMIN_PIN = "1234";

function doGet(e) {
  return HtmlService.createHtmlOutputFromFile('Index')
      .setTitle('QR Code Scanner')
      .setXFrameOptionsMode(HtmlService.XFrameOptionsMode.ALLOWALL);
}

/**
 * ฟังก์ชันหลักที่รับข้อมูลจาก Front-end
 */
function doPost(e) {
  var rawData = "";
  
  // ปรับปรุงการดึงข้อมูลให้ครอบคลุมทุกกรณีการส่ง
  if (e && e.postData && e.postData.contents) {
    rawData = e.postData.contents;
  } else if (e && e.parameter && e.parameter.jsonData) {
    rawData = e.parameter.jsonData;
  }

  if (!rawData) {
    return ContentService.createTextOutput(JSON.stringify({ success: false, message: "⚠️ ไม่พบข้อมูลที่ส่งมา (Empty Payload)" }))
                         .setMimeType(ContentService.MimeType.JSON);
  }

  try {
    var postData = JSON.parse(rawData);
    var action = postData.action || "scan";

    if (action === "getLogs") return ContentService.createTextOutput(JSON.stringify({ success: true, logs: getLogsData() })).setMimeType(ContentService.MimeType.JSON);
    if (action === "deleteLog") return ContentService.createTextOutput(JSON.stringify(deleteLogFromServer(postData.timestamp, postData.qrCode))).setMimeType(ContentService.MimeType.JSON);
    if (action === "adminReset") return ContentService.createTextOutput(JSON.stringify(adminResetLogStatus(postData.qrCode, postData.pin))).setMimeType(ContentService.MimeType.JSON);

    // ประมวลผลการสแกนปกติ
    var scanResult = updateAndFetchDetails(postData.qrData, postData.operatorName, postData.batchId);
    return ContentService.createTextOutput(JSON.stringify(scanResult)).setMimeType(ContentService.MimeType.JSON);

  } catch (err) {
    return ContentService.createTextOutput(JSON.stringify({ success: false, message: "🚨 Error: " + err.message })).setMimeType(ContentService.MimeType.JSON);
  }
}

// [เก็บฟังก์ชัน updateAndFetchDetails, getLogsData, deleteLogFromServer, adminResetLogStatus, checkAndTriggerDailyBackup ไว้เหมือนเดิมด้านล่างนี้ได้เลยครับ]
