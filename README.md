<html lang="ko">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>KT 법인회선 재약정 견적서 시스템</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <style>
        @page { size: A4; margin: 0; }
        * { 
            box-sizing: border-box; 
            font-family: 'Pretendard', -apple-system, BlinkMacSystemFont, 'Apple SD Gothic Neo', 'Malgun Gothic', '맑은 고딕', sans-serif;
            -webkit-user-select: none; -moz-user-select: none; -ms-user-select: none; user-select: none;
        }
        body {
            background-color: #f4f6f9;
            padding: 20px 0;
            margin: 0;
            display: flex; flex-direction: column; align-items: center; justify-content: center;
            -webkit-font-smoothing: antialiased; position: relative;
        }

        /* 🔒 보안 인증 오버레이 */
        .auth-overlay {
            position: fixed; top: 0; left: 0; width: 100vw; height: 100vh;
            background: rgba(15, 23, 42, 0.92); backdrop-filter: blur(8px);
            z-index: 999999; display: flex; justify-content: center; align-items: center;
        }
        .auth-card {
            background: #ffffff; width: 90%; max-width: 360px;
            border-radius: 16px; border-top: 6px solid #004b8d;
            box-shadow: 0 25px 50px -12px rgba(0, 75, 141, 0.35);
            padding: 28px 22px; text-align: center;
            animation: popIn 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275) forwards;
        }
        .auth-icon { font-size: 40px; margin-bottom: 8px; display: inline-block; }
        .auth-title { font-size: 19px; font-weight: 900; color: #004b8d; margin-bottom: 6px; letter-spacing: -0.5px; }
        .auth-sub { font-size: 12px; font-weight: 600; color: #64748b; margin-bottom: 18px; line-height: 1.4; }
        
        .auth-input {
            width: 100%; height: 44px; border: 2px solid #cbd5e1 !important;
            border-radius: 8px; font-size: 16px; font-weight: 700;
            text-align: center !important; letter-spacing: 4px; color: #1e293b !important;
            background: #f8fafc !important; margin-bottom: 12px; outline: none; transition: border-color 0.2s;
        }
        .auth-input:focus { border-color: #004b8d !important; background: #ffffff !important; }
        
        .auth-btn {
            width: 100%; height: 44px; background: #004b8d; color: #ffffff;
            font-size: 15px; font-weight: 800; border: none; border-radius: 8px;
            cursor: pointer; transition: background 0.2s, transform 0.1s;
        }
        .auth-btn:hover { background: #003666; }
        .auth-btn:active { transform: scale(0.98); }
        .auth-error { color: #dc2626; font-size: 12px; font-weight: 700; margin-top: 10px; display: none; }

        .auth-req-link {
            margin-top: 14px; font-size: 12px; color: #64748b; font-weight: 600; cursor: pointer; text-decoration: underline; transition: color 0.2s;
        }
        .auth-req-link:hover { color: #0284c7; }
        
        .req-form-area { display: none; margin-top: 14px; padding-top: 14px; border-top: 1px dashed #cbd5e1; }
        .req-input {
            width: 100%; height: 38px; border: 1px solid #cbd5e1 !important;
            border-radius: 6px; font-size: 13px; font-weight: 600;
            text-align: center !important; color: #1e293b !important;
            background: #f8fafc !important; margin-bottom: 8px; outline: none;
        }
        .req-input:focus { border-color: #0284c7 !important; background: #ffffff !important; }
        .req-submit-btn {
            width: 100%; height: 38px; background: #0284c7; color: #ffffff;
            font-size: 13.5px; font-weight: 800; border: none; border-radius: 6px; cursor: pointer; transition: background 0.2s;
        }
        .req-submit-btn:hover { background: #0369a1; }

        .shake { animation: shake 0.35s ease-in-out; }
        @keyframes shake {
            0%, 100% { transform: translateX(0); }
            20%, 60% { transform: translateX(-8px); }
            40%, 80% { transform: translateX(8px); }
        }

        .watermark-overlay {
            position: absolute; top: 50%; left: 50%;
            transform: translate(-50%, -50%) rotate(-25deg);
            font-size: 21px; font-weight: 900;
            color: rgba(0, 75, 141, 0.08) !important;
            white-space: nowrap; pointer-events: none; z-index: 999;
            letter-spacing: -0.5px; text-align: center;
        }

        .signature-badge {
            width: 794px; margin-bottom: 12px; padding: 10px 14px;
            background: #f0f9ff; border: 1px solid #bae6fd; border-radius: 8px;
            text-align: center; box-shadow: 0 2px 6px rgba(2, 132, 199, 0.06);
        }

        .red-alert-overlay {
            position: fixed; top: 0; left: 0; width: 100vw; height: 100vh;
            background: rgba(15, 23, 42, 0.7); backdrop-filter: blur(4px);
            z-index: 99999; display: none; justify-content: center; align-items: center;
            animation: fadeIn 0.2s ease-out;
        }
        .red-alert-card {
            background: #ffffff; width: 90%; max-width: 380px;
            border-radius: 16px; border-top: 6px solid #dc2626;
            box-shadow: 0 20px 30px rgba(220, 38, 38, 0.25);
            padding: 24px 20px; text-align: center;
            animation: popIn 0.25s cubic-bezier(0.175, 0.885, 0.32, 1.275) forwards;
        }
        .red-alert-icon { font-size: 40px; margin-bottom: 8px; display: inline-block; animation: pulse 1.2s infinite; }
        .red-alert-title { font-size: 18px; font-weight: 900; color: #dc2626; margin-bottom: 10px; letter-spacing: -0.5px; }
        .red-alert-msg {
            font-size: 15px; font-weight: 800; color: #1e293b; background: #fef2f2;
            border: 1px solid #fecaca; padding: 12px; border-radius: 8px; margin-bottom: 18px; line-height: 1.4;
        }
        .red-alert-btn {
            width: 100%; height: 42px; background: #dc2626; color: #ffffff;
            font-size: 14px; font-weight: 800; border: none; border-radius: 8px; cursor: pointer; transition: background 0.2s;
        }
        .red-alert-btn:hover { background: #b91c1c; }

        @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
        @keyframes popIn { from { transform: scale(0.85); opacity: 0; } to { transform: scale(1); opacity: 1; } }
        @keyframes pulse { 0%, 100% { transform: scale(1); } 50% { transform: scale(1.15); } }

        .toolbar-area {
            width: 100%; display: flex; justify-content: space-between; align-items: center;
            background-color: #f1f5f9; padding: 8px 12px; border-radius: 6px; margin-bottom: 12px; border: 1px solid #cbd5e1;
        }
        .toolbar-group { display: flex; gap: 6px; }
        .tool-btn {
            padding: 6px 12px; font-size: 11.5px; font-weight: 700; border-radius: 4px;
            cursor: pointer; border: 1px solid #cbd5e1; background-color: #ffffff; color: #334155; transition: all 0.2s;
        }
        .tool-btn:hover { background-color: #e2e8f0; }
        .tool-btn.btn-send { background-color: #0284c7; color: #ffffff; border-color: #0284c7; }
        .tool-btn.btn-send:hover { background-color: #0369a1; }
        .tool-btn.btn-save { background-color: #0d9488; color: #ffffff; border-color: #0d9488; }
        .tool-btn.btn-save:hover { background-color: #0f766e; }
        .tool-btn.btn-reset { background-color: #ef4444; color: #ffffff; border-color: #ef4444; }
        .tool-btn.btn-reset:hover { background-color: #dc2626; }

        .invoice-container {
            width: 794px; background-color: #ffffff; padding: 25px 35px;
            box-shadow: 0 4px 20px rgba(0,0,0,0.08); box-sizing: border-box;
            image-rendering: -webkit-optimize-contrast; margin: 0 auto; position: relative; overflow: hidden;
        }

        .invoice-header {
            display: flex; justify-content: space-between; align-items: center;
            margin-bottom: 15px; border-bottom: 3px solid #1c5280; padding-bottom: 6px;
        }
        .logo-area {
            font-size: 28px; font-weight: 900; color: #000000 !important;
            font-family: 'Arial Black', Impact, sans-serif; letter-spacing: -2px; line-height: 1;
        }
        .title-area {
            font-size: 24px; font-weight: 800; color: #004b8d !important;
            text-align: center; flex-grow: 1; letter-spacing: -0.5px; margin-right: 40px;
        }

        table {
            width: 100%; border-collapse: collapse; font-size: 11px; color: #000000 !important;
            margin-bottom: 8px; table-layout: fixed; box-sizing: border-box;
        }
        th, td {
            border: 1px solid #a0a0a0 !important; padding: 4px 5px; height: 26px;
            color: #000000 !important; vertical-align: middle !important; text-align: center !important;
            box-sizing: border-box; overflow: hidden;
        }
        th { background-color: #f1f5f9 !important; font-weight: 700 !important; color: #000000 !important; }

        .info-table th { width: 14%; }
        .info-table td { width: 36%; }

        .notice-container-table th { width: 11% !important; }
        .notice-container-table td { width: 89% !important; }

        .product-table th { background-color: #f1f5f9 !important; color: #000000 !important; font-weight: 700; font-size: 11px; }

        input[type="text"], input[type="date"], textarea {
            width: 100%; height: 100%; border: none !important; background-color: #f8fafc !important;
            padding: 0 4px; border-radius: 3px; font-size: 11px; font-family: inherit;
            color: #000000 !important; -webkit-text-fill-color: #000000 !important; opacity: 1 !important;
            font-weight: 600; box-sizing: border-box; outline: none; text-align: center !important; box-shadow: none !important;
        }

        textarea { text-align: left !important; padding: 6px !important; resize: none; height: 55px; line-height: 1.4; }

        .capture-text-node {
            font-size: 11px !important; font-weight: 600 !important; color: #000000 !important;
            text-align: center !important; width: 100% !important; display: block !important;
            line-height: 24px !important; height: 24px !important; white-space: nowrap !important;
            overflow: hidden !important; text-overflow: ellipsis !important; box-sizing: border-box !important;
        }

        input[type="date"] { color: transparent !important; -webkit-text-fill-color: transparent !important; }
        input[type="date"].has-value { color: #000000 !important; -webkit-text-fill-color: #000000 !important; }

        input:focus, textarea:focus { background-color: #e0f2fe !important; }
        input[readonly], .lock-cell {
            color: #475569 !important; -webkit-text-fill-color: #475569 !important;
            background-color: #f1f5f9 !important; font-weight: 600; cursor: not-allowed;
        }
        .blue-readonly { color: #004b8d !important; font-weight: 700; }
        .benefit-highlight {
            color: #d91414 !important; font-size: 13px; font-weight: 800;
            text-align: center !important; background-color: #fef2f2 !important; letter-spacing: 0.5px;
        }

        .total-row { background-color: #e2e8f0 !important; font-weight: 700; }

        .notice-text {
            font-size: 10.5px; color: #222222 !important; line-height: 1.45; font-weight: 500;
            text-align: left !important; white-space: normal !important; overflow: visible !important;
        }
        .bg-alert { background-color: #fffbeb !important; color: #b45309 !important; }

        .btn-area {
            margin-top: 15px; margin-bottom: 20px; width: 794px; display: flex; gap: 15px;
        }
        .download-btn {
            flex: 1; background-color: #004b8d; color: white; border: none; padding: 14px;
            font-size: 16px; font-weight: 700; cursor: pointer; border-radius: 6px;
            box-shadow: 0 4px 12px rgba(0, 61, 141, 0.25); transition: background 0.2s;
        }
        .download-btn:hover { background-color: #143757; }
        .pdf-btn { background-color: #10b981; box-shadow: 0 4px 12px rgba(16, 185, 129, 0.25); }
        .pdf-btn:hover { background-color: #047857; }

        .responsive-wrapper {
            width: 100%; overflow-x: auto; display: flex; flex-direction: column; align-items: center;
        }
    </style>
</head>
<body oncontextmenu="return false" onselectstart="return false" ondragstart="return false">

    <!-- 🔐 시스템 접속 보안 인증 모달 -->
    <div id="authOverlay" class="auth-overlay">
        <div class="auth-card" id="authCard">
            <div class="auth-icon">🔒</div>
            <div class="auth-title">시스템 보안 인증</div>
            <div class="auth-sub">인가된 사용자만 접속 가능합니다.<br/>비밀번호를 입력하세요.</div>
            
            <input type="password" id="authPasswordInput" class="auth-input" placeholder="••••" maxlength="20" autocomplete="current-password" />
            <button type="button" class="auth-btn" id="authSubmitBtn">확인 (접속하기)</button>
            <div id="authErrorMsg" class="auth-error">⚠️ 비밀번호가 일치하지 않습니다. (기본 암호: 0729)</div>
            
            <div class="auth-req-link" id="toggleReqFormBtn">비밀번호가 없으신가요? (암호 신청하기)</div>

            <div id="reqFormArea" class="req-form-area">
                <div style="font-size: 11.5px; font-weight: 700; color: #0284c7; margin-bottom: 8px;">🔑 비밀번호 발급 신청</div>
                <input type="text" id="reqUserName" class="req-input" placeholder="신청자 성함 입력" maxlength="20" />
                <input type="text" id="reqUserPhone" class="req-input" placeholder="연락처 (예: 010-1234-5678)" maxlength="15" />
                <button type="button" class="req-submit-btn" id="reqSubmitBtn">신청 문자/알림 전송</button>
            </div>
        </div>
    </div>

    <!-- 🚨 경고 모달 레이어 -->
    <div id="redAlertOverlay" class="red-alert-overlay">
        <div class="red-alert-card">
            <div class="red-alert-icon">🚨</div>
            <div class="red-alert-title">경 고 (WARNING)</div>
            <div class="red-alert-msg">무단 복제 및 불펌을 금지합니다.</div>
            <button class="red-alert-btn" onclick="closeRedAlert()">확인 (닫기)</button>
        </div>
    </div>

    <div class="responsive-wrapper">
        
        <!-- 최상단 공식 시그니처 배지 -->
        <div class="signature-badge">
            <div style="font-size:11px; color:#0284c7; font-weight:800; letter-spacing:1px; margin-bottom:2px;">TELECOM ESTIMATE SYSTEM</div>
            <div style="font-size:13px; color:#1e293b; font-weight:700;">
                🔒 본 시스템은 <span style="color:#0284c7;">(주)케이티엠앤에스</span> 공식 견적 산출 전용 시스템입니다.
            </div>
        </div>

        <!-- ================= [서식: 법인회선 재약정 견적서] ================= -->
        <div class="invoice-container" id="capture-area-renewal">
            <div class="watermark-overlay">(주)케이티엠앤에스 견적서 - 무단 복사 및 사용 금지</div>
            
            <div class="toolbar-area no-print-target">
                <div class="toolbar-group">
                    <button class="tool-btn" onclick="recalcForm()">🔄 합계 새로고침</button>
                    <button class="tool-btn btn-reset" onclick="resetActiveTabForm()">♻️ 입력 내용 초기화</button>
                </div>
                <div class="toolbar-group">
                    <button class="tool-btn btn-send" onclick="sendQuoteDataGas()">📩 DB 적재 & 알림 전송</button>
                    <button class="tool-btn btn-save" onclick="saveCurrentEstimateData()">💾 최근 작성 저장</button>
                    <button class="tool-btn" onclick="loadSavedEstimateData()">📂 불러오기</button>
                </div>
            </div>

            <div class="invoice-header">
                <div class="logo-area">kt</div>
                <div class="title-area">법인회선 재약정 견적서</div>
            </div>

            <table class="info-table">
                <tr>
                    <th>견적일자</th>
                    <td><input type="text" class="invoice-date blue-readonly" readonly /></td>
                    <th>사업자번호</th>
                    <td><input type="text" value="120-87-09780" class="lock-cell" readonly /></td>
                </tr>
                <tr>
                    <th>업체명</th>
                    <td><input type="text" value=" 귀하" class="client-name" id="renewal-client-name" onfocus="clearGuidance(this)" onblur="restoreGuidance(this)" /></td>
                    <th>회사명</th>
                    <td><input type="text" value="(주)케이티엠앤에스" class="lock-cell" readonly /></td>
                </tr>
                <tr>
                    <th>사업자번호</th>
                    <td><input type="text" id="renewal-biz-num" placeholder="고객 사업자번호 입력" /></td>
                    <th>대표자명</th>
                    <td><input type="text" value="박성열" class="lock-cell" readonly /></td>
                </tr>
                <tr>
                    <th>총 제공되는 혜택</th>
                    <td class="benefit-highlight" id="total-benefits-renewal">₩0</td>
                    <th>주소</th>
                    <td><input type="text" value="경기도 성남시 분당구 불정로 90 KT타워" class="lock-cell" readonly /></td>
                </tr>
                <tr>
                    <th>수수료</th>
                    <td><input type="text" id="fee-renewal" value="0" oninput="runBenefitCalculationsRenewal(this)" /></td>
                    <th>업종</th>
                    <td><input type="text" value="정보통신업, 통신기기" class="lock-cell" readonly /></td>
                </tr>
                <tr>
                    <th>통합사은품</th>
                    <td><input type="text" id="gift-renewal" value="0" oninput="runBenefitCalculationsRenewal(this)" /></td>
                    <th>담당부서</th>
                    <td><input type="text" value="KT M&S 동부법인지사 부산센터" class="lock-cell" readonly /></td>
                </tr>
                <tr>
                    <th rowspan="2">구비서류</th>
                    <td rowspan="2" class="notice-text lock-cell" style="background-color: #fafafa; font-size: 10px; line-height: 1.45; text-align: center !important;">
                        <span class="capture-text-node" style="font-size: 10px !important;">대표자신분증, 사업자등록증, 통장사본</span>
                    </td>
                    <th>담당자</th>
                    <td><input type="text" id="manager-name-renewal" class="blue-readonly" value="윤혜진 과장" readonly /></td>
                </tr>
                <tr>
                    <th>연락처</th>
                    <td><input type="text" id="manager-phone-renewal" class="blue-readonly" value="010-9969-1904" readonly /></td>
                </tr>
                <tr>
                    <th>견적유효기간</th>
                    <td class="notice-text lock-cell" style="background-color: #ffffff; font-weight: bold; color: #004b8d !important; text-align: center !important;">
                        <span class="capture-text-node" style="color: #004b8d !important; font-weight: bold !important;">견적서 제출일로부터 30일 이내</span>
                    </td>
                    <th>이메일주소</th>
                    <td><input type="text" id="manager-email-renewal" class="blue-readonly" value="ktmns1248591@gmail.com" readonly /></td>
                </tr>
            </table>

            <table class="product-table" id="product-table-renewal">
                <thead>
                    <tr>
                        <th style="width:18%">설치장소</th>
                        <th style="width:15%">가입 상품</th>
                        <th style="width:16%">약정만료기간</th>
                        <th style="width:13%">상품 요금제</th>
                        <th style="width:11%">청구금액</th>
                        <th style="width:12%">갱신시 요금</th>
                        <th style="width:15%">기타 (요금변동)</th>
                    </tr>
                </thead>
                <tbody>
                    <script>
                        for(let i=0; i<15; i++) {
                            document.write(`
                            <tr>
                                <td><input type="text" /></td>
                                <td><input type="text" /></td>
                                <td><input type="date" onchange="checkDateValue(this)" /></td>
                                <td><input type="text" /></td>
                                <td><input type="text" class="calc-charge-renewal" oninput="runCalculationsRenewal(this)" /></td>
                                <td><input type="text" class="calc-renew-renewal" oninput="runCalculationsRenewal(this)" /></td>
                                <td><input type="text" class="calc-diff-renewal" readonly placeholder="-" /></td>
                            </tr>`);
                        }
                    </script>
                    <tr class="total-row">
                        <td colspan="4" class="text-center">최종합계</td>
                        <td id="total-charge-renewal">0</td>
                        <td id="total-renew-renewal">0</td>
                        <td id="total-diff-renewal">0</td>
                    </tr>
                </tbody>
            </table>

            <table class="notice-container-table">
                <tr>
                    <th style="color: #d91414 !important; background-color: #fef2f2 !important; text-align: center !important;">필수 안내</th>
                    <td class="notice-text bg-alert" style="padding: 10px; font-weight: 700;">
                        • 수수료 지급 : 계약/개통 완료 ➔ 익월 말 ➔ 법인 대표자(또는 명의자) 휴대폰으로 모바일 상품권 발송<br />
                        • 사은품 지급 : 계약/개통 완료 ➔ 1주 이내 ➔ 법인 대표자(또는 명의자) 휴대폰으로 모바일 상품권 발송<br />
                        • <span style="color:#d91414;">[지급 제한 조건]</span> 수수료 및 통합 사은품은 명함/서류에 기재된 확인 가능한 본인 전화번호로만 발송됩니다.
                    </td>
                </tr>
            </table>

            <table class="notice-container-table">
                <tr>
                    <th style="text-align: center !important;">유의 사항</th>
                    <td class="notice-text" style="padding: 10px; background-color: #fafafa; color: #555 !important;">
                        1. 상기 법인 회선 재약정 기준 KT 정식 약정 기간은 총 3년(36개월)이며, 기타 상세 세부사항은 KT 이용약관에 준합니다.<br />
                        2. 재약정 시점에 따라 일부 가입 상품(기업용 TV 등)의 (구)요금제가 (신)요금제로 변경될 수 있습니다.<br />
                        3. 현장 설비 이상 및 장애 발생 시 즉시 KT 고객센터(국번없이 100번)를 통해 접수하시거나 담당부서로 상담 바랍니다.
                    </td>
                </tr>
            </table>

            <table class="notice-container-table">
                <tr>
                    <th style="text-align: center !important; background-color: #f1f5f9;">메모 사항</th>
                    <td style="padding: 0; background-color: #f8fafc;">
                        <textarea placeholder="여기에 특이사항이나 추가 협의 내용을 입력하세요."></textarea>
                    </td>
                </tr>
            </table>
        </div>

        <!-- 하단 전용 다운로드 버튼 영역 -->
        <div class="btn-area">
            <button class="download-btn" onclick="generateActiveInvoiceImage('jpg')">견적서 JPG 다운로드</button>
            <button class="download-btn pdf-btn" onclick="generateActiveInvoiceImage('pdf')">견적서 PDF 다운로드</button>
        </div>
    </div>

    <script>
        const SYSTEM_PASSWORD = "0729";
        const GAS_URL = "https://script.google.com/macros/s/AKfycbzriskJha8aL9cnErvdImPwMBxLi690oyLCUgrTBHJHcvHiWlNvGwU3ferdftgx-sml/exec";

        window.onload = function() {
            setTodayDate();
            calculateBenefitsRenewal();
            calculateTableTotalsRenewal();
            initKeyLock();
            initAuthEvents();
        }

        function initAuthEvents() {
            const authInput = document.getElementById('authPasswordInput');
            const submitBtn = document.getElementById('authSubmitBtn');
            const toggleBtn = document.getElementById('toggleReqFormBtn');
            const reqSubmitBtn = document.getElementById('reqSubmitBtn');

            if (authInput) {
                authInput.value = '';
                setTimeout(() => authInput.focus(), 100);
                authInput.addEventListener('keydown', function(e) {
                    if (e.key === 'Enter' || e.keyCode === 13) {
                        e.preventDefault();
                        validatePassword();
                    }
                });
            }

            if (submitBtn) {
                submitBtn.addEventListener('click', function(e) {
                    e.preventDefault();
                    validatePassword();
                });
            }

            if (toggleBtn) {
                toggleBtn.addEventListener('click', function(e) {
                    e.preventDefault();
                    toggleRequestForm();
                });
            }

            if (reqSubmitBtn) {
                reqSubmitBtn.addEventListener('click', function(e) {
                    e.preventDefault();
                    submitPasswordRequest();
                });
            }

            const phoneInput = document.getElementById('reqUserPhone');
            if (phoneInput) {
                phoneInput.addEventListener('keydown', function(e) {
                    if (e.key === 'Enter' || e.keyCode === 13) {
                        e.preventDefault();
                        submitPasswordRequest();
                    }
                });
            }
        }

        function validatePassword() {
            const input = document.getElementById('authPasswordInput');
            const errMsg = document.getElementById('authErrorMsg');
            const card = document.getElementById('authCard');
            const enteredVal = String(input.value).trim();

            if (enteredVal === SYSTEM_PASSWORD) {
                document.getElementById('authOverlay').style.display = 'none';
                errMsg.style.display = 'none';
            } else {
                errMsg.style.display = 'block';
                card.classList.add('shake');
                input.value = '';
                input.focus();
                setTimeout(() => { card.classList.remove('shake'); }, 400);
            }
        }

        function toggleRequestForm() {
            const area = document.getElementById('reqFormArea');
            if (area.style.display === 'block') {
                area.style.display = 'none';
            } else {
                area.style.display = 'block';
                document.getElementById('reqUserName').focus();
            }
        }

        function submitPasswordRequest() {
            const nameInput = document.getElementById('reqUserName');
            const phoneInput = document.getElementById('reqUserPhone');
            const btn = document.getElementById('reqSubmitBtn');
            const name = nameInput.value.trim();
            const phone = phoneInput.value.trim();

            if (!name) { alert('신청자 성함을 입력해 주세요.'); nameInput.focus(); return; }
            if (!phone) { alert('연락처를 입력해 주세요.'); phoneInput.focus(); return; }

            btn.disabled = true;
            btn.innerText = '전송 중...';

            const payload = {
                type: 'PASSWORD_REQUEST',
                userName: name,
                userPhone: phone,
                requestDate: new Date().toLocaleString('ko-KR', { timeZone: 'Asia/Seoul' }),
                consultDay: `[암호신청] ${name} (${phone})`
            };

            fetch(GAS_URL, {
                method: 'POST', mode: 'no-cors',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            }).then(() => {
                alert(`[신청 완료]\n${name}님의 암호 발급 신청이 담당자에게 전달되었습니다.`);
                nameInput.value = ''; phoneInput.value = '';
                document.getElementById('reqFormArea').style.display = 'none';
                btn.disabled = false; btn.innerText = '신청 문자/알림 전송';
            }).catch(err => {
                alert('신청 전송 중 오류가 발생했습니다.');
                btn.disabled = false; btn.innerText = '신청 문자/알림 전송';
            });
        }

        function showRedAlert() { document.getElementById('redAlertOverlay').style.display = 'flex'; }
        function closeRedAlert() { document.getElementById('redAlertOverlay').style.display = 'none'; }

        function initKeyLock() {
            document.addEventListener('contextmenu', function(e) { e.preventDefault(); showRedAlert(); });
            document.addEventListener('dragstart', e => e.preventDefault());
            document.addEventListener('selectstart', function(e) {
                if (e.target && (e.target.id === 'authPasswordInput' || e.target.id === 'reqUserName' || e.target.id === 'reqUserPhone')) return true;
                e.preventDefault();
            });

            document.addEventListener('keydown', function(e) {
                if (e.target && (e.target.id === 'authPasswordInput' || e.target.id === 'reqUserName' || e.target.id === 'reqUserPhone')) return;
                if (e.keyCode === 123 || (e.ctrlKey && e.shiftKey && (e.keyCode === 73 || e.keyCode === 74 || e.keyCode === 67)) || (e.ctrlKey && (e.keyCode === 85 || e.keyCode === 83 || e.keyCode === 67 || e.keyCode === 65))) {
                    e.preventDefault(); e.stopPropagation(); showRedAlert(); return false;
                }
            });
        }

        function setTodayDate() {
            const today = new Date();
            const yyyy = today.getFullYear();
            const mm = String(today.getMonth() + 1).padStart(2, '0');
            const dd = String(today.getDate()).padStart(2, '0');
            document.querySelectorAll('.invoice-date').forEach(el => { el.value = `${yyyy}-${mm}-${dd}`; });
        }

        function clearGuidance(el) { if(el.value === " 귀하") { el.value = ""; } }
        function restoreGuidance(el) {
            if(el.value.trim() === "") { el.value = " 귀하"; } 
            else if(!el.value.endsWith(" 귀하")) { el.value = el.value + " 귀하"; }
        }

        function checkDateValue(el) {
            if (el.value) { el.classList.add('has-value'); } else { el.classList.remove('has-value'); }
        }

        function sendQuoteDataGas() {
            const userPhone = "01099691904";
            let payload = { quoteType: 'renewal', userPhone: userPhone, consultDay: '견적서 즉시발행' };
            fetch(GAS_URL, {
                method: 'POST', mode: 'no-cors',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            }).then(() => alert('구글 시트 DB 적재 및 알림 전송 완료')).catch(err => alert('전송 오류: ' + err));
        }

        function recalcForm() {
            calculateBenefitsRenewal();
            calculateTableTotalsRenewal();
            alert('모든 금액 합계가 새로고침되었습니다.');
        }

        function resetActiveTabForm() {
            if(!confirm('현재 견적서 내용을 초기화하시겠습니까?')) return;
            const targetContainer = document.getElementById('capture-area-renewal');
            targetContainer.querySelectorAll('input[type="text"]:not([readonly]), textarea').forEach(inp => {
                if(!inp.classList.contains('invoice-date')) { inp.value = ''; }
            });
            document.getElementById('fee-renewal').value = '0';
            document.getElementById('gift-renewal').value = '0';
            calculateBenefitsRenewal();
            calculateTableTotalsRenewal();
        }

        function saveCurrentEstimateData() {
            const targetContainer = document.getElementById('capture-area-renewal');
            const formData = {};
            targetContainer.querySelectorAll('input, textarea').forEach((el, index) => {
                if(!el.readOnly) {
                    formData[el.id ? `id_${el.id}` : `idx_${index}`] = el.value;
                }
            });
            localStorage.setItem('saved_estimate_renewal', JSON.stringify(formData));
            alert('입력 내용이 브라우저에 저장되었습니다.');
        }

        function loadSavedEstimateData() {
            const savedData = localStorage.getItem('saved_estimate_renewal');
            if(!savedData) { alert('저장된 데이터가 없습니다.'); return; }
            if(!confirm('불러오시겠습니까?')) return;

            const formData = JSON.parse(savedData);
            const targetContainer = document.getElementById('capture-area-renewal');
            targetContainer.querySelectorAll('input, textarea').forEach((el, index) => {
                if(!el.readOnly) {
                    const kId = `id_${el.id}`, kIdx = `idx_${index}`;
                    if(el.id && formData[kId] !== undefined) el.value = formData[kId];
                    else if(formData[kIdx] !== undefined) el.value = formData[kIdx];
                }
            });
            calculateBenefitsRenewal();
            calculateTableTotalsRenewal();
        }

        function runCalculationsRenewal(el) { formatNumberInput(el); calculateRowDifferenceRenewal(el.closest('tr')); calculateTableTotalsRenewal(); }
        function calculateRowDifferenceRenewal(row) {
            const c = row.querySelector('.calc-charge-renewal'), r = row.querySelector('.calc-renew-renewal'), d = row.querySelector('.calc-diff-renewal');
            if(!c || !r || !d) return;
            const charge = parseInt(c.value.replace(/,/g, '')) || 0, renew = parseInt(r.value.replace(/,/g, '')) || 0;
            if(c.value === "" && r.value === "") { d.value = ""; return; }
            const diff = renew - charge;
            d.value = (diff >= 0 ? "+" : "") + diff.toLocaleString();
        }
        function runBenefitCalculationsRenewal(el) { formatNumberInput(el, "0"); calculateBenefitsRenewal(); }
        function calculateBenefitsRenewal() {
            const f = parseInt(document.getElementById('fee-renewal').value.replace(/,/g, '')) || 0;
            const g = parseInt(document.getElementById('gift-renewal').value.replace(/,/g, '')) || 0;
            document.getElementById('total-benefits-renewal').innerText = '₩' + (f + g).toLocaleString();
        }
        function calculateTableTotalsRenewal() {
            let tc = 0, tr = 0;
            document.querySelectorAll('.calc-charge-renewal').forEach(i => { tc += parseInt(i.value.replace(/,/g, '')) || 0; });
            document.querySelectorAll('.calc-renew-renewal').forEach(i => { tr += parseInt(i.value.replace(/,/g, '')) || 0; });
            document.getElementById('total-charge-renewal').innerText = tc.toLocaleString();
            document.getElementById('total-renew-renewal').innerText = tr.toLocaleString();
            const td = tr - tc;
            document.getElementById('total-diff-renewal').innerText = (tc === 0 && tr === 0) ? "0" : ((td >= 0 ? "+" : "") + td.toLocaleString());
        }

        function formatNumberInput(el, defaultVal = "") {
            let cursorPosition = el.selectionStart, originalLength = el.value.length;
            let value = el.value.replace(/[^0-9]/g, '');
            el.value = (value !== "") ? parseInt(value).toLocaleString() : defaultVal;
            let newLength = el.value.length;
            el.setSelectionRange(cursorPosition + (newLength - originalLength), cursorPosition + (newLength - originalLength));
        }

        function generateActiveInvoiceImage(format) {
            const originArea = document.getElementById('capture-area-renewal');
            if (document.activeElement) document.activeElement.blur();

            const cloneArea = originArea.cloneNode(true);
            cloneArea.querySelectorAll('.no-print-target').forEach(btn => btn.style.display = 'none');

            cloneArea.style.position = 'absolute';
            cloneArea.style.top = '-9999px';
            cloneArea.style.left = '-9999px';
            cloneArea.style.width = '794px';
            cloneArea.style.backgroundColor = '#ffffff';
            cloneArea.style.display = 'block';
            
            document.body.appendChild(cloneArea);

            const originInputs = originArea.querySelectorAll('input, textarea');
            const cloneInputs = cloneArea.querySelectorAll('input, textarea');

            originInputs.forEach((originInput, idx) => {
                const cloneInput = cloneInputs[idx];
                if (!cloneInput) return;

                let text = (originInput.type === 'date' || originInput.tagName === 'TEXTAREA') ? (originInput.value || ' ') : (originInput.value || originInput.placeholder || ' ');

                const textNode = document.createElement('span');
                textNode.className = 'capture-text-node';
                
                if (originInput.classList.contains('blue-readonly')) {
                    textNode.style.color = '#004b8d'; 
                    textNode.style.fontWeight = 'bold';
                }

                if (originInput.tagName === 'TEXTAREA') {
                    textNode.style.whiteSpace = 'pre-wrap'; 
                    textNode.style.textAlign = 'left'; 
                    textNode.style.padding = '6px'; 
                    textNode.style.display = 'block';
                    textNode.style.lineHeight = '1.4';
                    textNode.style.height = 'auto';
                }

                textNode.innerText = text;
                if (cloneInput.parentNode) cloneInput.parentNode.replaceChild(textNode, cloneInput);
            });

            setTimeout(() => {
                html2canvas(cloneArea, {
                    scale: 3,
                    useCORS: true,
                    backgroundColor: '#ffffff',
                    logging: false,
                    width: 794,
                    windowWidth: 794
                }).then(canvas => {
                    const imageData = canvas.toDataURL('image/jpeg', 1.0);
                    const fileName = '법인회선_재약정_견적서';

                    if (format === 'jpg') {
                        const link = document.createElement('a');
                        link.href = imageData;
                        link.download = `${fileName}.jpg`;
                        link.click();
                    } else if (format === 'pdf') {
                        const { jsPDF } = window.jspdf;
                        const pdf = new jsPDF('p', 'mm', 'a4');
                        const imgProps = pdf.getImageProperties(imageData);
                        const pdfWidth = pdf.internal.pageSize.getWidth();
                        const pdfHeight = (imgProps.height * pdfWidth) / imgProps.width;
                        pdf.addImage(imageData, 'JPEG', 0, 0, pdfWidth, pdfHeight);
                        pdf.save(`${fileName}.pdf`);
                    }
                    
                    document.body.removeChild(cloneArea);
                }).catch(err => {
                    if (document.body.contains(cloneArea)) document.body.removeChild(cloneArea);
                    alert("파일 생성 오류: " + err);
                });
            }, 100);
        }
    </script>
</body>
</html>
