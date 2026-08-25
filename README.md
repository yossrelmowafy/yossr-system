<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>نظام الحضور والاشتراكات - مستر يسر الموافي</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
    <style>
        body {
            background-image: linear-gradient(rgba(15, 23, 42, 0.8), rgba(15, 23, 42, 0.8)), url('https://images.unsplash.com/photo-1497633762265-9d179a990aa6?q=80&w=1920&auto=format&fit=crop');
            background-size: cover;
            background-position: center;
            background-attachment: fixed;
        }
        @media print {
            body { background: white !important; color: black !important; }
            .no-print { display: none !important; }
        }
    </style>
</head>
<body class="font-sans min-h-screen">

    <!-- شاشة تسجيل الدخول للمشرف -->
    <div id="login-screen" class="min-h-screen flex items-center justify-center p-4">
        <div class="bg-white/95 backdrop-blur p-8 rounded-2xl shadow-2xl w-full max-w-md">
            <h2 class="text-2xl font-bold text-center text-blue-600 mb-2">لوحة تحكم المشرف</h2>
            <p class="text-center text-gray-500 text-sm mb-6">مستر يسر الموافي</p>
            <div id="login-error" class="hidden bg-red-100 text-red-700 p-3 rounded mb-4 text-sm text-center"></div>
            <form onsubmit="handleLogin(event)" class="space-y-4">
                <div>
                    <label class="block text-gray-700 mb-2 font-medium">اسم المستخدم</label>
                    <input type="text" id="username" required class="w-full px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500">
                </div>
                <div>
                    <label class="block text-gray-700 mb-2 font-medium">كلمة المرور</label>
                    <input type="password" id="password" required class="w-full px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500">
                </div>
                <button type="submit" class="w-full bg-blue-600 text-white py-2.5 rounded-lg hover:bg-blue-700 transition font-medium">دخول لوحة التحكم</button>
            </form>
        </div>
    </div>

    <!-- لوحة التحكم الرئيسية للمشرف -->
    <div id="dashboard" class="hidden min-h-screen flex flex-col">
        <header class="bg-blue-600/90 backdrop-blur text-white p-4 shadow-md flex justify-between items-center no-print">
            <h1 class="text-xl font-bold">إدارة الحضور والاشتراكات - مستر يسر الموافي</h1>
            <button onclick="logout()" class="bg-red-500 px-4 py-1.5 rounded text-sm hover:bg-red-600 transition font-medium">تسجيل خروج</button>
        </header>

        <main class="flex-1 p-4 max-w-6xl w-full mx-auto space-y-6">
            <!-- أزرار التنقل -->
            <div class="flex gap-2 bg-white/90 backdrop-blur p-2 rounded-xl shadow overflow-x-auto no-print">
                <button onclick="switchTab('requests')" id="btn-requests" class="flex-1 py-2 px-4 rounded-lg bg-blue-600 text-white font-medium transition whitespace-nowrap">الطلبات الواردة (<span id="req-count">0</span>)</button>
                <button onclick="switchTab('search')" id="btn-search" class="flex-1 py-2 px-4 rounded-lg text-gray-700 font-medium transition whitespace-nowrap">بحث شامل عن طالب</button>
                <button onclick="switchTab('students')" id="btn-students" class="flex-1 py-2 px-4 rounded-lg text-gray-700 font-medium transition whitespace-nowrap">إدارة الطلاب والـ QR</button>
                <button onclick="switchTab('subscriptions')" id="btn-subscriptions" class="flex-1 py-2 px-4 rounded-lg text-gray-700 font-medium transition whitespace-nowrap">الاشتراكات والماليات</button>
                <button onclick="switchTab('reports')" id="btn-reports" class="flex-1 py-2 px-4 rounded-lg text-gray-700 font-medium transition whitespace-nowrap">سجل الحضور والغياب</button>
            </div>

            <!-- 1. طلبات الحضور والدفع المعلقة -->
            <div id="tab-requests" class="bg-white/95 backdrop-blur p-6 rounded-xl shadow-xl space-y-4">
                <h3 class="text-lg font-bold text-gray-800">مراجعة طلبات الطلاب (حضور أو مدفوعات)</h3>
                <div class="overflow-x-auto">
                    <table class="w-full text-right border-collapse">
                        <thead>
                            <tr class="bg-gray-100 text-gray-700">
                                <th class="p-3 border">اسم الطالب</th>
                                <th class="p-3 border">نوع الطلب</th>
                                <th class="p-3 border">التفاصيل / الشهر</th>
                                <th class="p-3 border">وقت الإرسال</th>
                                <th class="p-3 border">قرارك</th>
                            </tr>
                        </thead>
                        <tbody id="requests-table-body">
                            <tr><td colspan="5" class="text-center p-4 text-gray-400">لا توجد طلبات معلقة حالياً</td></tr>
                        </tbody>
                    </table>
                </div>
            </div>

            <!-- 2. قسم البحث الشامل عن الطالب (بالاختيار من القائمة) -->
            <div id="tab-search" class="bg-white/95 backdrop-blur p-6 rounded-xl shadow-xl space-y-4 hidden">
                <h3 class="text-lg font-bold text-gray-800">🔍 بحث شامل عن تفاصيل الطالب</h3>
                <div class="flex gap-2 items-center">
                    <select id="master-search-select" class="flex-1 px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 bg-white">
                        <option value="">-- اختر اسم الطالب من القائمة --</option>
                    </select>
                    <button onclick="executeMasterSearch()" class="bg-blue-600 text-white px-6 py-2 rounded-lg hover:bg-blue-700 transition font-medium">عرض الملف</button>
                </div>

                <!-- نتيجة البحث وعرض الملف -->
                <div id="master-search-result" class="hidden space-y-4 border-t pt-4">
                    <div class="bg-blue-50 p-4 rounded-xl border border-blue-200">
                        <div class="flex justify-between items-start flex-wrap gap-2">
                            <div>
                                <h4 id="m-res-name" class="text-xl font-bold text-blue-900">--</h4>
                                <p class="text-sm text-gray-600">الرقم التعريفي: <span id="m-res-code" class="font-bold text-gray-800">--</span> | الموبايل: <span id="m-res-phone" class="font-bold text-gray-800">--</span></p>
                            </div>
                        </div>
                    </div>

                    <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                        <!-- اشتراكات الطالب -->
                        <div class="bg-gray-50 p-4 rounded-xl border">
                            <h5 class="font-bold text-gray-800 mb-2">الاشتراكات والماليات الخاصة به:</h5>
                            <div id="m-res-subs" class="space-y-2 text-sm"></div>
                        </div>
                        <!-- سجل الحضور للطالب -->
                        <div class="bg-gray-50 p-4 rounded-xl border">
                            <h5 class="font-bold text-gray-800 mb-2">سجل الحضور والغياب:</h5>
                            <div id="m-res-attendance" class="space-y-1 text-sm max-h-40 overflow-y-auto"></div>
                        </div>
                    </div>
                </div>

                <div id="master-no-result" class="hidden text-center text-red-500 py-6 font-semibold">
                    الرجاء اختيار طالب صالح لعرض بياناته.
                </div>
            </div>

            <!-- 3. إدارة الطلاب والـ QR -->
            <div id="tab-students" class="bg-white/95 backdrop-blur p-6 rounded-xl shadow-xl space-y-4 hidden">
                <div class="flex flex-col md:flex-row justify-between gap-4 items-center">
                    <h3 class="text-lg font-bold text-gray-800">إضافة طالب جديد وتوليد الـ QR</h3>
                    <input type="text" id="search-student" oninput="renderStudents()" placeholder="بحث سريع عن طالب..." class="px-4 py-1.5 border rounded-lg text-sm w-full md:w-64">
                </div>
                <form onsubmit="addStudent(event)" class="grid grid-cols-1 md:grid-cols-3 gap-4">
                    <input type="text" id="student-name" placeholder="اسم الطالب الكامل" required class="px-4 py-2 border rounded-lg">
                    <input type="text" id="student-code" placeholder="الرقم التعريفي (مثال: YOSS-101)" required class="px-4 py-2 border rounded-lg">
                    <input type="tel" id="student-phone" placeholder="رقم التليفون" required class="px-4 py-2 border rounded-lg">
                    <button type="submit" class="md:col-span-3 bg-green-600 text-white py-2 rounded-lg hover:bg-green-700 transition font-medium">حفظ الطالب وتوليد الـ QR</button>
                </form>

                <hr class="my-4">

                <div class="overflow-x-auto">
                    <table class="w-full text-right border-collapse">
                        <thead>
                            <tr class="bg-gray-100 text-gray-700">
                                <th class="p-3 border">اسم الطالب</th>
                                <th class="p-3 border">الرقم التعريفي</th>
                                <th class="p-3 border">رقم التليفون</th>
                                <th class="p-3 border">رابط صفحة الطالب</th>
                                <th class="p-3 border">إجراءات</th>
                            </tr>
                        </thead>
                        <tbody id="students-table-body"></tbody>
                    </table>
                </div>
            </div>

            <!-- 4. قسم الاشتراكات والماليات -->
            <div id="tab-subscriptions" class="bg-white/95 backdrop-blur p-6 rounded-xl shadow-xl space-y-4 hidden">
                <div class="flex flex-wrap justify-between items-center gap-3 bg-red-50 p-4 rounded-xl border border-red-200">
                    <div>
                        <h4 class="font-bold text-red-800 text-sm">نهاية الشهر وبدء شهر جديد؟</h4>
                        <p class="text-xs text-red-600">تصفير الحضور والاشتراكات مع الاحتفاظ بأسماء وأكواد الطلاب.</p>
                    </div>
                    <button onclick="resetNewMonth()" class="bg-red-600 text-white px-4 py-2 rounded-lg text-sm font-bold hover:bg-red-700 transition">تصفير حسابات الشهر وبدء شهر جديد</button>
                </div>

                <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                    <div class="bg-blue-50 border border-blue-200 p-4 rounded-xl text-center">
                        <p class="text-xs text-blue-600 font-medium">إجمالي المستهدف</p>
                        <p id="stat-total" class="text-2xl font-bold text-blue-900">0 ج</p>
                    </div>
                    <div class="bg-green-50 border border-green-200 p-4 rounded-xl text-center">
                        <p class="text-xs text-green-600 font-medium">إجمالي المحصل</p>
                        <p id="stat-paid" class="text-2xl font-bold text-green-900">0 ج</p>
                    </div>
                    <div class="bg-red-50 border border-red-200 p-4 rounded-xl text-center">
                        <p class="text-xs text-red-600 font-medium">إجمالي المتبقي (ديون)</p>
                        <p id="stat-remaining" class="text-2xl font-bold text-red-900">0 ج</p>
                    </div>
                </div>

                <div class="flex flex-col md:flex-row justify-between gap-4 items-center">
                    <h3 class="text-lg font-bold text-gray-800">متابعة وتعديل اشتراكات الطلاب</h3>
                    <div class="flex gap-2 w-full md:w-auto">
                        <select id="filter-sub-month" onchange="renderSubscriptions()" class="px-3 py-1.5 border rounded-lg text-sm">
                            <option value="">كل الشهور</option>
                            <option value="سبتمبر (9)">سبتمبر (9)</option>
                            <option value="أكتوبر (10)">أكتوبر (10)</option>
                            <option value="نوفمبر (11)">نوفمبر (11)</option>
                            <option value="ديسمبر (12)">ديسمبر (12)</option>
                            <option value="يناير (1)">يناير (1)</option>
                            <option value="فبراير (2)">فبراير (2)</option>
                            <option value="مارس (3)">مارس (3)</option>
                            <option value="أبريل (4)">أبريل (4)</option>
                            <option value="مايو (5)">مايو (5)</option>
                            <option value="يونيو (6)">يونيو (6)</option>
                            <option value="يوليو (7)">يوليو (7)</option>
                            <option value="أغسطس (8)">أغسطس (8)</option>
                        </select>
                        <button onclick="exportTableToExcel('subscriptions-table', 'subscriptions_report.xls')" class="bg-emerald-600 text-white px-4 py-1.5 rounded-lg text-sm hover:bg-emerald-700 transition font-medium whitespace-nowrap">تصدير لـ Excel</button>
                    </div>
                </div>

                <form onsubmit="addSubscription(event)" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-7 gap-3">
                    <select id="sub-student-name" required class="px-3 py-2 border rounded-lg">
                        <option value="">اختر الطالب</option>
                    </select>
                    <select id="sub-month" required class="px-3 py-2 border rounded-lg">
                        <option value="">اختر الشهر</option>
                        <option value="سبتمبر (9)">سبتمبر (9)</option>
                        <option value="أكتوبر (10)">أكتوبر (10)</option>
                        <option value="نوفمبر (11)">نوفمبر (11)</option>
                        <option value="ديسمبر (12)">ديسمبر (12)</option>
                        <option value="يناير (1)">يناير (1)</option>
                        <option value="فبراير (2)">فبراير (2)</option>
                        <option value="مارس (3)">مارس (3)</option>
                        <option value="أبريل (4)">أبريل (4)</option>
                        <option value="مايو (5)">مايو (5)</option>
                        <option value="يونيو (6)">يونيو (6)</option>
                        <option value="يوليو (7)">يوليو (7)</option>
                        <option value="أغسطس (8)">أغسطس (8)</option>
                    </select>
                    <input type="text" id="sub-category" placeholder="المرحلة/الصنف" required class="px-3 py-2 border rounded-lg">
                    <input type="number" id="sub-total" placeholder="إجمالي المبلغ" required class="px-3 py-2 border rounded-lg">
                    <input type="number" id="sub-paid" placeholder="المدفوع" required class="px-3 py-2 border rounded-lg">
                    <input type="text" id="sub-time" placeholder="ميعاد الحصة (مثال: السبت 4)" required class="px-3 py-2 border rounded-lg">
                    <button type="submit" class="bg-blue-600 text-white py-2 rounded-lg hover:bg-blue-700 transition font-medium">حفظ الاشتراك</button>
                </form>

                <hr class="my-4">

                <div class="overflow-x-auto">
                    <table id="subscriptions-table" class="w-full text-right border-collapse">
                        <thead>
                            <tr class="bg-gray-100 text-gray-700">
                                <th class="p-3 border">اسم الطالب</th>
                                <th class="p-3 border">الشهر</th>
                                <th class="p-3 border">المرحلة</th>
                                <th class="p-3 border">الإجمالي</th>
                                <th class="p-3 border">المدفوع</th>
                                <th class="p-3 border">المتبقي</th>
                                <th class="p-3 border">الميعاد</th>
                                <th class="p-3 border no-print">إجراءات</th>
                            </tr>
                        </thead>
                        <tbody id="subscriptions-table-body"></tbody>
                    </table>
                </div>
            </div>

            <!-- 5. تقرير الحضور والغياب -->
            <div id="tab-reports" class="bg-white/95 backdrop-blur p-6 rounded-xl shadow-xl space-y-4 hidden">
                <div class="flex justify-between items-center no-print flex-wrap gap-2">
                    <h3 class="text-lg font-bold text-gray-800">سجل الحضور والغياب الشهري للطلاب</h3>
                    <div class="flex gap-2 flex-wrap">
                        <button onclick="markAbsenteesForToday()" class="bg-orange-600 text-white px-4 py-1.5 rounded text-sm hover:bg-orange-700 transition font-medium">تسجيل غياب اليوم لمن لم يحضر</button>
                        <button onclick="exportTableToExcel('attendance-table', 'attendance_report.xls')" class="bg-emerald-600 text-white px-4 py-1.5 rounded text-sm hover:bg-emerald-700 transition font-medium">تصدير لـ Excel</button>
                        <button onclick="window.print()" class="bg-gray-700 text-white px-4 py-1.5 rounded text-sm hover:bg-gray-800 transition font-medium">طباعة التقرير</button>
                    </div>
                </div>
                <div class="overflow-x-auto">
                    <table id="attendance-table" class="w-full text-right border-collapse">
                        <thead>
                            <tr class="bg-gray-100 text-gray-700">
                                <th class="p-3 border">اسم الطالب</th>
                                <th class="p-3 border">الرقم التعريفي</th>
                                <th class="p-3 border">تاريخ ووقت السجل</th>
                                <th class="p-3 border">الحالة</th>
                            </tr>
                        </thead>
                        <tbody id="attendance-table-body"></tbody>
                    </table>
                </div>
            </div>
        </main>
    </div>

    <!-- نافذة الطالب المخصصة -->
    <div id="student-portal" class="hidden min-h-screen flex items-center justify-center p-4">
        <div class="bg-white/95 backdrop-blur p-8 rounded-2xl shadow-2xl w-full max-w-md text-center space-y-4">
            <h2 class="text-xl font-bold text-blue-600">بوابة الطالب الشخصية</h2>
            <div id="portal-content" class="space-y-3"></div>
        </div>
    </div>

    <!-- نافذة تعديل الاشتراك -->
    <div id="edit-modal" class="hidden fixed inset-0 bg-black/60 backdrop-blur-sm flex items-center justify-center p-4">
        <div class="bg-white p-6 rounded-2xl shadow-2xl space-y-4 max-w-md w-full text-right">
            <h3 class="font-bold text-lg text-gray-800">تعديل بيانات الاشتراك</h3>
            <input type="hidden" id="edit-index">
            <div class="space-y-3">
                <div>
                    <label class="block text-xs text-gray-600 mb-1">الشهر</label>
                    <select id="edit-month" class="w-full px-3 py-2 border rounded">
                        <option value="سبتمبر (9)">سبتمبر (9)</option>
                        <option value="أكتوبر (10)">أكتوبر (10)</option>
                        <option value="نوفمبر (11)">نوفمبر (11)</option>
                        <option value="ديسمبر (12)">ديسمبر (12)</option>
                        <option value="يناير (1)">يناير (1)</option>
                        <option value="فبراير (2)">فبراير (2)</option>
                        <option value="مارس (3)">مارس (3)</option>
                        <option value="أبريل (4)">أبريل (4)</option>
                        <option value="مايو (5)">مايو (5)</option>
                        <option value="يونيو (6)">يونيو (6)</option>
                        <option value="يوليو (7)">يوليو (7)</option>
                        <option value="أغسطس (8)">أغسطس (8)</option>
                    </select>
                </div>
                <div>
                    <label class="block text-xs text-gray-600 mb-1">المرحلة / الصنف</label>
                    <input type="text" id="edit-category" class="w-full px-3 py-2 border rounded">
                </div>
                <div>
                    <label class="block text-xs text-gray-600 mb-1">إجمالي المبلغ</label>
                    <input type="number" id="edit-total" class="w-full px-3 py-2 border rounded">
                </div>
                <div>
                    <label class="block text-xs text-gray-600 mb-1">المبلغ المدفوع</label>
                    <input type="number" id="edit-paid" class="w-full px-3 py-2 border rounded">
                </div>
                <div>
                    <label class="block text-xs text-gray-600 mb-1">ميعاد الحصة</label>
                    <input type="text" id="edit-time" class="w-full px-3 py-2 border rounded">
                </div>
            </div>
            <div class="flex gap-2 pt-2">
                <button onclick="saveEditedSubscription()" class="flex-1 bg-green-600 text-white py-2 rounded font-medium">حفظ التعديل</button>
                <button onclick="closeEditModal()" class="flex-1 bg-gray-400 text-white py-2 rounded font-medium">إلغاء</button>
            </div>
        </div>
    </div>

    <!-- نافذة الـ QR -->
    <div id="qr-modal" class="hidden fixed inset-0 bg-black/60 backdrop-blur-sm flex items-center justify-center p-4">
        <div class="bg-white p-6 rounded-2xl shadow-2xl text-center space-y-4 max-w-sm w-full">
            <h3 id="modal-student-name" class="font-bold text-lg text-gray-800"></h3>
            <div id="qrcode" class="flex justify-center p-2 bg-white border rounded"></div>
            <p class="text-xs text-gray-500">اجعل الطالب يمسح هذا الكود بكاميرا هاتفه وقت الحصة</p>
            <button onclick="closeQrModal()" class="w-full bg-blue-600 text-white py-2 rounded-lg font-medium">إغلاق</button>
        </div>
    </div>

    <script>
        const ADMIN_USER = "yossr_elmowafy";
        const ADMIN_PASS = "23122002YY";

        let students = JSON.parse(localStorage.getItem('students')) || [];
        let subscriptions = JSON.parse(localStorage.getItem('subscriptions')) || [];
        let pendingRequests = JSON.parse(localStorage.getItem('pendingRequests')) || [];
        let attendanceLog = JSON.parse(localStorage.getItem('attendanceLog')) || [];

        window.onload = function() {
            const urlParams = new URLSearchParams(window.location.search);
            const studentCode = urlParams.get('student');

            if (studentCode) {
                document.getElementById('login-screen').classList.add('hidden');
                document.getElementById('student-portal').classList.remove('hidden');
                renderStudentPortal(studentCode);
            }
        };

        function handleLogin(e) {
            e.preventDefault();
            if (document.getElementById('username').value === ADMIN_USER && document.getElementById('password').value === ADMIN_PASS) {
                document.getElementById('login-screen').classList.add('hidden');
                document.getElementById('dashboard').classList.remove('hidden');
                renderAll();
            } else {
                const err = document.getElementById('login-error');
                err.textContent = "بيانات الدخول غير صحيحة!";
                err.classList.remove('hidden');
            }
        }

        function logout() { location.reload(); }

        function switchTab(tab) {
            ['requests', 'search', 'students', 'subscriptions', 'reports'].forEach(t => {
                document.getElementById(`tab-${t}`).classList.add('hidden');
                document.getElementById(`btn-${t}`).className = "flex-1 py-2 px-4 rounded-lg text-gray-700 font-medium transition whitespace-nowrap";
            });
            document.getElementById(`tab-${tab}`).classList.remove('hidden');
            document.getElementById(`btn-${tab}`).className = "flex-1 py-2 px-4 rounded-lg bg-blue-600 text-white font-medium transition whitespace-nowrap";
        }

        function addStudent(e) {
            e.preventDefault();
            const name = document.getElementById('student-name').value;
            const code = document.getElementById('student-code').value;
            const phone = document.getElementById('student-phone').value;

            if (students.some(s => s.code === code)) {
                alert("الرقم التعريفي مستخدم مسبقاً!");
                return;
            }

            students.push({ name, code, phone });
            localStorage.setItem('students', JSON.stringify(students));
            document.getElementById('student-name').value = '';
            document.getElementById('student-code').value = '';
            document.getElementById('student-phone').value = '';
            renderAll();
            alert("تم إضافة الطالب وتوليد الـ QR بنجاح!");
        }

        function deleteStudent(index) {
            students.splice(index, 1);
            localStorage.setItem('students', JSON.stringify(students));
            renderAll();
        }

        function resetNewMonth() {
            if (confirm("هل أنت متأكد من تصفير حسابات الشهر الحالي وسجلات الحضور والغياب؟ (سيتم الاحتفاظ بالطلاب وأكوادهم والـ QR فقط).")) {
                subscriptions = [];
                attendanceLog = [];
                pendingRequests = [];
                localStorage.setItem('subscriptions', JSON.stringify(subscriptions));
                localStorage.setItem('attendanceLog', JSON.stringify(attendanceLog));
                localStorage.setItem('pendingRequests', JSON.stringify(pendingRequests));
                renderAll();
                alert("تم تصفير الشهر بنجاح وبدء شهر جديد.");
            }
        }

        // خاصية البحث الشامل باختيار اسم الطالب من القائمة المنسدلة
        function executeMasterSearch() {
            const selectVal = document.getElementById('master-search-select').value;
            const resultBox = document.getElementById('master-search-result');
            const noResBox = document.getElementById('master-no-result');

            if (!selectVal) {
                resultBox.classList.add('hidden');
                noResBox.classList.remove('hidden');
                return;
            }

            const student = students.find(s => s.name === selectVal);

            if (student) {
                noResBox.classList.add('hidden');
                resultBox.classList.remove('hidden');

                document.getElementById('m-res-name').textContent = student.name;
                document.getElementById('m-res-code').textContent = student.code;
                document.getElementById('m-res-phone').textContent = student.phone || 'غير متوفر';

                const studentSubs = subscriptions.filter(sub => sub.name === student.name);
                const subsContainer = document.getElementById('m-res-subs');
                subsContainer.innerHTML = '';

                if (studentSubs.length > 0) {
                    studentSubs.forEach(sub => {
                        subsContainer.innerHTML += `
                            <div class="bg-white p-2.5 rounded border shadow-sm">
                                <p class="font-bold text-purple-700">${sub.month} - ${sub.category}</p>
                                <div class="flex justify-between text-xs mt-1">
                                    <span>الإجمالي: <b class="text-blue-600">${sub.total}ج</b></span>
                                    <span>المدفوع: <b class="text-green-600">${sub.paid}ج</b></span>
                                    <span>المتبقي: <b class="text-red-600">${sub.remaining}ج</b></span>
                                </div>
                            </div>
                        `;
                    });
                } else {
                    subsContainer.innerHTML = `<p class="text-gray-500 text-xs">لا توجد اشتراكات مسجلة لهذا الطالب.</p>`;
                }

                const studentAtt = attendanceLog.filter(att => att.name === student.name);
                const attContainer = document.getElementById('m-res-attendance');
                attContainer.innerHTML = '';

                if (studentAtt.length > 0) {
                    studentAtt.forEach(att => {
                        const badge = att.status === 'غائب' ? '<span class="text-red-600 font-bold">غائب</span>' : '<span class="text-green-600 font-bold">حاضر</span>';
                        attContainer.innerHTML += `
                            <div class="bg-white p-2 rounded border flex justify-between items-center text-xs">
                                <span>${att.date}</span>
                                <span>${badge}</span>
                            </div>
                        `;
                    });
                } else {
                    attContainer.innerHTML = `<p class="text-gray-500 text-xs">لا توجد سجلات حضور مسجلة.</p>`;
                }

            } else {
                resultBox.classList.add('hidden');
                noResBox.classList.remove('hidden');
            }
        }

        function addSubscription(e) {
            e.preventDefault();
            const name = document.getElementById('sub-student-name').value;
            const month = document.getElementById('sub-month').value;
            const category = document.getElementById('sub-category').value;
            const total = parseFloat(document.getElementById('sub-total').value);
            const paid = parseFloat(document.getElementById('sub-paid').value);
            const remaining = total - paid;
            const time = document.getElementById('sub-time').value;

            subscriptions.push({ name, month, category, total, paid, remaining, time });
            localStorage.setItem('subscriptions', JSON.stringify(subscriptions));
            
            document.getElementById('sub-category').value = '';
            document.getElementById('sub-total').value = '';
            document.getElementById('sub-paid').value = '';
            document.getElementById('sub-time').value = '';
            renderSubscriptions();
            alert("تم حفظ بيانات الاشتراك بنجاح!");
        }

        function deleteSubscription(index) {
            subscriptions.splice(index, 1);
            localStorage.setItem('subscriptions', JSON.stringify(subscriptions));
            renderSubscriptions();
        }

        function openEditModal(index) {
            const sub = subscriptions[index];
            document.getElementById('edit-index').value = index;
            document.getElementById('edit-month').value = sub.month;
            document.getElementById('edit-category').value = sub.category;
            document.getElementById('edit-total').value = sub.total;
            document.getElementById('edit-paid').value = sub.paid;
            document.getElementById('edit-time').value = sub.time;
            document.getElementById('edit-modal').classList.remove('hidden');
        }

        function closeEditModal() { document.getElementById('edit-modal').classList.add('hidden'); }

        function saveEditedSubscription() {
            const index = document.getElementById('edit-index').value;
            const total = parseFloat(document.getElementById('edit-total').value);
            const paid = parseFloat(document.getElementById('edit-paid').value);

            subscriptions[index].month = document.getElementById('edit-month').value;
            subscriptions[index].category = document.getElementById('edit-category').value;
            subscriptions[index].total = total;
            subscriptions[index].paid = paid;
            subscriptions[index].remaining = total - paid;
            subscriptions[index].time = document.getElementById('edit-time').value;

            localStorage.setItem('subscriptions', JSON.stringify(subscriptions));
            closeEditModal();
            renderSubscriptions();
            alert("تم تعديل البيانات بنجاح!");
        }

        function exportTableToExcel(tableID, filename = '') {
            let dataType = 'application/vnd.ms-excel';
            let tableSelect = document.getElementById(tableID);
            let tableHTML = tableSelect.outerHTML.replace(/<button[^>]*>.*?<\/button>/g, '');
            filename = filename ? filename : 'excel_data.xls';
            let downloadLink = document.createElement("a");
            document.body.appendChild(downloadLink);
            downloadLink.href = 'data:' + dataType + ', ' + encodeURIComponent('\ufeff' + tableHTML);
            downloadLink.download = filename;
            downloadLink.click();
        }

        function renderAll() {
            renderStudents();
            renderSubscriptions();
            renderRequests();
            renderAttendance();
            renderSearchSelect();
            document.getElementById('req-count').textContent = pendingRequests.length;
        }

        // تعبئة قائمة البحث وأسماء الطلاب
        function renderSearchSelect() {
            const searchSelect = document.getElementById('master-search-select');
            const currentSelected = searchSelect.value;
            searchSelect.innerHTML = '<option value="">-- اختر اسم الطالب من القائمة --</option>';
            
            students.forEach(s => {
                searchSelect.innerHTML += `<option value="${s.name}">${s.name} (${s.code})</option>`;
            });
            searchSelect.value = currentSelected;
        }

        function renderStudents() {
            const tbody = document.getElementById('students-table-body');
            const select = document.getElementById('sub-student-name');
            const searchVal = document.getElementById('search-student') ? document.getElementById('search-student').value.toLowerCase() : '';
            
            tbody.innerHTML = '';
            select.innerHTML = '<option value="">اختر الطالب</option>';

            const filteredStudents = students.filter(s => s.name.toLowerCase().includes(searchVal) || s.code.toLowerCase().includes(searchVal));

            students.forEach((s) => {
                select.innerHTML += `<option value="${s.name}">${s.name}</option>`;
            });

            // إصلاح رابط الـ QR ليعمل أينما كان (حتى لو مرفوع على استضافة مجانية)
            const baseUrl = window.location.href.split('?')[0];

            filteredStudents.forEach((s) => {
                const realIndex = students.indexOf(s);
                const url = `${baseUrl}?student=${s.code}`;
                tbody.innerHTML += `
                    <tr class="border-b hover:bg-gray-50">
                        <td class="p-3 border">${s.name}</td>
                        <td class="p-3 border">${s.code}</td>
                        <td class="p-3 border">${s.phone || '-'}</td>
                        <td class="p-3 border"><button onclick="showQrCode('${s.code}', '${s.name}', '${url}')" class="bg-indigo-50 text-indigo-600 px-2.5 py-1 rounded text-xs font-medium hover:bg-indigo-100">عرض الـ QR</button></td>
                        <td class="p-3 border"><button onclick="deleteStudent(${realIndex})" class="text-red-600 hover:underline text-sm font-medium">حذف</button></td>
                    </tr>
                `;
            });
            renderSearchSelect();
        }

        function renderSubscriptions() {
            const tbody = document.getElementById('subscriptions-table-body');
            const filterMonth = document.getElementById('filter-sub-month').value;
            tbody.innerHTML = '';

            let totalSum = 0, paidSum = 0, remainingSum = 0;
            const filteredSubs = subscriptions.filter(sub => !filterMonth || sub.month === filterMonth);

            filteredSubs.forEach(sub => {
                totalSum += sub.total;
                paidSum += sub.paid;
                remainingSum += sub.remaining;
            });

            document.getElementById('stat-total').textContent = totalSum + ' ج';
            document.getElementById('stat-paid').textContent = paidSum + ' ج';
            document.getElementById('stat-remaining').textContent = remainingSum + ' ج';

            if (filteredSubs.length === 0) {
                tbody.innerHTML = `<tr><td colspan="8" class="text-center p-4 text-gray-400">لا توجد اشتراكات مسجلة لهذا الشهر</td></tr>`;
                return;
            }

            filteredSubs.forEach((sub) => {
                const realIndex = subscriptions.indexOf(sub);
                tbody.innerHTML += `
                    <tr class="border-b hover:bg-gray-50">
                        <td class="p-3 border">${sub.name}</td>
                        <td class="p-3 border"><span class="bg-purple-100 text-purple-700 px-2 py-0.5 rounded text-xs font-bold">${sub.month}</span></td>
                        <td class="p-3 border">${sub.category}</td>
                        <td class="p-3 border font-bold text-blue-600">${sub.total} ج</td>
                        <td class="p-3 border text-green-600">${sub.paid} ج</td>
                        <td class="p-3 border font-bold text-red-600">${sub.remaining} ج</td>
                        <td class="p-3 border">${sub.time}</td>
                        <td class="p-3 border space-x-2 space-x-reverse no-print">
                            <button onclick="openEditModal(${realIndex})" class="bg-blue-50 text-blue-600 px-2.5 py-1 rounded text-xs font-medium hover:bg-blue-100">تعديل</button>
                            <button onclick="deleteSubscription(${realIndex})" class="text-red-600 hover:underline text-xs font-medium">حذف</button>
                        </td>
                    </tr>
                `;
            });
        }

        function showQrCode(code, name, url) {
            document.getElementById('modal-student-name').textContent = name;
            document.getElementById('qrcode').innerHTML = "";
            new QRCode(document.getElementById("qrcode"), { text: url, width: 180, height: 180 });
            document.getElementById('qr-modal').classList.remove('hidden');
        }

        function closeQrModal() { document.getElementById('qr-modal').classList.add('hidden'); }

        function renderStudentPortal(code) {
            const student = students.find(s => s.code === code);
            const portal = document.getElementById('portal-content');
            if (!student) {
                portal.innerHTML = `<p class="text-red-600 font-bold">عذراً، هذا الكود غير مسجل.</p>`;
                return;
            }

            const studentSubs = subscriptions.filter(s => s.name === student.name);
            let paymentAlertHtml = '';
            if (studentSubs.length === 0 || studentSubs.some(s => s.remaining > 0)) {
                paymentAlertHtml = `
                    <div class="bg-amber-100 border-r-4 border-amber-500 text-amber-800 p-3 rounded text-xs font-bold text-right">
                        ⚠️ تنبيه: يجب دفع اشتراك الشهر وتأكيده عبر القائمة أدناه.
                    </div>
                `;
            }

            let subsHtml = studentSubs.length ? studentSubs.map(sub => `
                <div class="bg-white p-2.5 rounded shadow-sm text-xs space-y-1 mb-2 border">
                    <p class="font-bold text-purple-700">${sub.month} - ${sub.category}</p>
                    <div class="flex justify-around text-center">
                        <span>الإجمالي: <b class="text-blue-600">${sub.total}ج</b></span>
                        <span>المدفوع: <b class="text-green-600">${sub.paid}ج</b></span>
                        <span>المتبقي: <b class="text-red-600">${sub.remaining}ج</b></span>
                    </div>
                    <p class="text-gray-500 text-center">الميعاد: ${sub.time}</p>
                </div>
            `).join('') : '<p class="text-xs text-gray-500">لا توجد اشتراكات مسجلة بعد لهذا الشهر</p>';

            portal.innerHTML = `
                ${paymentAlertHtml}
                <div class="bg-blue-50 p-4 rounded-lg text-right space-y-2">
                    <div>
                        <p class="text-gray-600 text-xs">اسم الطالب:</p>
                        <p class="font-bold text-lg text-blue-900">${student.name}</p>
                    </div>
                    <div class="pt-2 border-t">
                        <p class="text-gray-600 text-xs mb-1">تفاصيل الاشتراكات:</p>
                        ${subsHtml}
                    </div>
                </div>

                <div class="space-y-3 pt-2">
                    <button onclick="submitAttendanceRequest('${student.code}', '${student.name}')" class="w-full bg-green-600 text-white py-2.5 rounded-lg font-bold hover:bg-green-700 transition">تسجيل حضور حصة اليوم</button>
                    
                    <div class="bg-gray-50 p-3 rounded-lg border space-y-2 text-right">
                        <label class="block text-xs font-bold text-gray-700">إبلاغ المستر بدفع الاشتراك:</label>
                        <select id="pay-month" class="w-full px-3 py-1.5 border rounded text-sm mb-1">
                            <option value="سبتمبر (9)">سبتمبر (9)</option>
                            <option value="أكتوبر (10)">أكتوبر (10)</option>
                            <option value="نوفمبر (11)">نوفمبر (11)</option>
                            <option value="ديسمبر (12)">ديسمبر (12)</option>
                            <option value="يناير (1)">يناير (1)</option>
                            <option value="فبراير (2)">فبراير (2)</option>
                            <option value="مارس (3)">مارس (3)</option>
                            <option value="أبريل (4)">أبريل (4)</option>
                            <option value="مايو (5)">مايو (5)</option>
                            <option value="يونيو (6)">يونيو (6)</option>
                            <option value="يوليو (7)">يوليو (7)</option>
                            <option value="أغسطس (8)">أغسطس (8)</option>
                        </select>
                        <div class="flex gap-2">
                            <input type="number" id="pay-amount" placeholder="المبلغ المدفوع" class="w-full px-3 py-1.5 border rounded text-sm">
                            <button onclick="submitPaymentRequest('${student.name}')" class="bg-blue-600 text-white px-4 py-1.5 rounded text-sm font-medium hover:bg-blue-700 whitespace-nowrap">تم الدفع (إرسال للمستر)</button>
                        </div>
                    </div>
                </div>
                <div id="portal-msg" class="text-sm font-medium"></div>
            `;
        }

        function submitAttendanceRequest(code, name) {
            const dateStr = new Date().toLocaleDateString('ar-EG');
            const timeStr = new Date().toLocaleTimeString('ar-EG');
            const fullDateTime = `${dateStr} - ${timeStr}`;

            if (pendingRequests.some(r => r.code === code && r.date.includes(dateStr) && r.type === 'حضور')) {
                document.getElementById('portal-msg').innerHTML = `<span class="text-orange-600">أرسلت طلب حضور مسبقاً اليوم!</span>`;
                return;
            }
            pendingRequests.push({ type: 'حضور', name, code, details: 'حضور الحصة', date: fullDateTime });
            localStorage.setItem('pendingRequests', JSON.stringify(pendingRequests));
            document.getElementById('portal-msg').innerHTML = `<span class="text-green-600">تم تسجيل حضورك بنجاح!</span>`;
        }

        function submitPaymentRequest(name) {
            const amount = parseFloat(document.getElementById('pay-amount').value);
            const month = document.getElementById('pay-month').value;
            if (!amount || amount <= 0) {
                alert("يرجى إدخال مبلغ صحيح!");
                return;
            }
            const dateStr = new Date().toLocaleDateString('ar-EG');
            const timeStr = new Date().toLocaleTimeString('ar-EG');
            const fullDateTime = `${dateStr} - ${timeStr}`;

            pendingRequests.push({ type: 'دفع', name, code: '-', details: `دفع ${amount} ج لـ (${month})`, amount: amount, month: month, date: fullDateTime });
            localStorage.setItem('pendingRequests', JSON.stringify(pendingRequests));
            document.getElementById('portal-msg').innerHTML = `<span class="text-green-600">تم إرسال إشعار الدفع للمستر للمراجعة.</span>`;
        }

        function renderRequests() {
            const tbody = document.getElementById('requests-table-body');
            tbody.innerHTML = '';
            if (pendingRequests.length === 0) {
                tbody.innerHTML = `<tr><td colspan="5" class="text-center p-4 text-gray-400">لا توجد طلبات معلقة حالياً</td></tr>`;
                return;
            }
            pendingRequests.forEach((req, index) => {
                tbody.innerHTML += `
                    <tr class="border-b hover:bg-gray-50">
                        <td class="p-3 border">${req.name}</td>
                        <td class="p-3 border"><span class="px-2 py-1 rounded text-xs ${req.type === 'حضور' ? 'bg-green-100 text-green-700' : 'bg-blue-100 text-blue-700'}">${req.type}</span></td>
                        <td class="p-3 border">${req.details}</td>
                        <td class="p-3 border">${req.date}</td>
                        <td class="p-3 border space-x-2 space-x-reverse">
                            <button onclick="approveRequest(${index})" class="bg-green-600 text-white px-3 py-1 rounded text-xs font-medium hover:bg-green-700">قبول</button>
                            <button onclick="rejectRequest(${index})" class="bg-red-600 text-white px-3 py-1 rounded text-xs font-medium hover:bg-red-700">رفض</button>
                        </td>
                    </tr>
                `;
            });
        }

        function approveRequest(index) {
            const req = pendingRequests.splice(index, 1)[0];
            if (req.type === 'حضور') {
                attendanceLog.unshift({ status: 'حاضر', name: req.name, code: req.code, date: req.date });
                localStorage.setItem('attendanceLog', JSON.stringify(attendanceLog));
            } else if (req.type === 'دفع') {
                const sub = subscriptions.find(s => s.name === req.name && s.month === req.month);
                if (sub) {
                    sub.paid += req.amount;
                    sub.remaining = sub.total - sub.paid;
                    localStorage.setItem('subscriptions', JSON.stringify(subscriptions));
                } else {
                    subscriptions.push({ name: req.name, month: req.month, category: 'اشتراك شهري', total: req.amount, paid: req.amount, remaining: 0, time: 'يُحدد لاحقاً' });
                    localStorage.setItem('subscriptions', JSON.stringify(subscriptions));
                }
            }
            localStorage.setItem('pendingRequests', JSON.stringify(pendingRequests));
            renderAll();
        }

        function rejectRequest(index) {
            pendingRequests.puts ? pendingRequests.splice(index, 1) : pendingRequests.splice(index, 1);
            localStorage.setItem('pendingRequests', JSON.stringify(pendingRequests));
            renderAll();
        }

        function markAbsenteesForToday() {
            const todayStr = new Date().toLocaleDateString('ar-EG');
            const timeStr = new Date().toLocaleTimeString('ar-EG');
            const fullDateTime = `${todayStr} - ${timeStr}`;
            let addedCount = 0;

            students.forEach(student => {
                const alreadyRecorded = attendanceLog.some(a => a.name === student.name && a.date.includes(todayStr));
                if (!alreadyRecorded) {
                    attendanceLog.unshift({ status: 'غائب', name: student.name, code: student.code, date: fullDateTime });
                    addedCount++;
                }
            });

            localStorage.setItem('attendanceLog', JSON.stringify(attendanceLog));
            renderAttendance();
            alert(`تم رصد الغياب بنجاح لـ ${addedCount} طالب.`);
        }

        function renderAttendance() {
            const tbody = document.getElementById('attendance-table-body');
            tbody.innerHTML = '';
            if (attendanceLog.length === 500 && attendanceLog.length === 0) { // Safety
                tbody.innerHTML = `<tr><td colspan="4" class="text-center p-4 text-gray-400">لا توجد سجلات حضور</td></tr>`;
                return;
            }
            if (attendanceLog.length === 0) {
                tbody.innerHTML = `<tr><td colspan="4" class="text-center p-4 text-gray-400">لا توجد سجلات حضور أو غياب بعد</td></tr>`;
                return;
            }
            attendanceLog.forEach(a => {
                const statusBadge = a.status === 'غائب' 
                    ? '<span class="bg-red-100 text-red-700 px-2.5 py-1 rounded text-xs font-bold">غائب</span>' 
                    : '<span class="bg-green-100 text-green-700 px-2.5 py-1 rounded text-xs font-bold">حاضر</span>';

                tbody.innerHTML += `
                    <tr class="border-b hover:bg-gray-50">
                        <td class="p-3 border font-medium">${a.name}</td>
                        <td class="p-3 border">${a.code}</td>
                        <td class="p-3 border">${a.date}</td>
                        <td class="p-3 border">${statusBadge}</td>
                    </tr>
                `;
            });
        }
    </script>
</body>
</html>
