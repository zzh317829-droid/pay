<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>延时费缴纳系统 (云开发版)</title>
    <style>
        *{ margin:0; padding:0; box-sizing:border-box; font-family:"Microsoft YaHei", sans-serif; }
        body{ background:#f5f7fa; min-height:100vh; display:flex; align-items:center; justify-content:center; padding:20px; }
        .page{ width:100%; max-width:500px; background:#fff; border-radius:12px; box-shadow:0 2px 12px rgba(0,0,0,0.1); padding:35px 30px; display:none; }
        .page.active{ display:block; }
        h2{ text-align:center; color:#333; margin-bottom:30px; }
        h3{ color:#333; margin:25px 0 15px; font-size:16px; display:flex; justify-content:space-between; align-items:center; }
        .form-item{ margin-bottom:20px; }
        label{ display:block; color:#666; margin-bottom:8px; }
        input{ width:100%; height:44px; padding:0 15px; border:1px solid #ddd; border-radius:6px; }
        .btn-group{ display:flex; gap:12px; margin-top:10px; }
        button{ width:100%; height:45px; border:none; border-radius:6px; font-size:16px; cursor:pointer; transition:all 0.2s; }
        button:disabled{ background:#ccc!important; cursor:not-allowed; }
        .btn-primary{ background:#409eff; color:#fff; }
        .btn-success{ background:#07c160; color:#fff; }
        .btn-default{ background:#909399; color:#fff; }
        .btn-warning{ background:#e6a23c; color:#fff; }
        .btn-danger{ background:#f56c6c; color:#fff; }
        .btn-small{ width:auto; height:30px; padding:0 10px; font-size:12px; }
        .amount-box{ text-align:center; padding:30px 0; margin:20px 0; background:#f0f9ff; border-radius:8px; }
        .amount-title{ color:#666; font-size:15px; margin-bottom:10px; }
        .amount-num{ color:#f56c6c; font-size:36px; font-weight:700; }
        .pay-info{ background:#f5f7fa; padding:15px; border-radius:6px; margin:20px 0; }
        .pay-info p{ color:#666; margin:8px 0; }
        .toast{ position:fixed; top:50%; left:50%; transform:translate(-50%,-50%); background:rgba(0,0,0,0.75); color:#fff; padding:12px 20px; border-radius:6px; z-index:9999; display:none; }
        .toast.show{ display:block; }
        .home-btn{ margin-bottom:20px; height:55px; font-size:17px; }
        .user-list,.order-list{ max-height:400px; overflow-y:auto; margin:15px 0; border:1px solid #ddd; border-radius:6px; }
        .user-item,.order-item{ display:flex; justify-content:space-between; align-items:center; padding:10px 15px; border-bottom:1px solid #f0f0f0; }
        .user-item:last-child,.order-item:last-child{ border-bottom:none; }
        .user-item:hover,.order-item:hover{ background:#fafa; }
        .item-content{ flex:1; }
        .item-btn-group{ display:flex; gap:8px; flex-shrink:0; }
        .status-tag{ padding:3px 8px; border-radius:4px; font-size:12px; }
        .status-wait{ background:#fdf6ec; color:#e6a23c; }
        .status-success{ background:#f0f9ff; color:#07c160; }
        .pay-modal{ position:fixed; top:0; left:0; width:100%; height:100%; background:rgba(0,0,0,0.7); display:none; align-items:center; justify-content:center; z-index:9998; }
        .pay-modal-box{ background:#fff; border-radius:12px; padding:30px; max-width:350px; width:90%; text-align:center; }
        .pay-code-img{ width:250px; height:250px; margin:0 auto 20px; display:block; border:1px solid #eee; border-radius:8px; }
        .search-box{ display:flex; gap:10px; margin-bottom:15px; }
        .search-box input{ margin:0; }
        .search-box button{ width:120px; flex-shrink:0; }
        .data-count{ color:#999; font-size:12px; font-weight:normal; }
        .batch-btn-group{ display:flex; gap:10px; margin-bottom:15px; flex-wrap:wrap; }
        .batch-btn-group button{ flex:1; height:35px; font-size:14px; min-width:120px; }
        .loading{ text-align:center; padding:20px; color:#999; }
    </style>
    <!-- 引入腾讯云开发Web SDK -->
    <script src="https://cdn.jsdelivr.net/npm/@cloudbase/js-sdk@latest/dist/index.min.js"></script>
</head>
<body>
    <!-- 首页 -->
    <div id="homePage" class="page active">
        <h2>延时费缴纳系统 (云开发版)</h2>
        <p style="text-align:center;color:#999;margin-bottom:30px;">官方缴费通道 | 安全支付 | 云端存储</p>
        <button class="home-btn btn-primary" onclick="go('loginPage')">用户缴费入口</button>
        <button class="home-btn btn-default" onclick="go('adminLoginPage')">管理后台</button>
    </div>

    <!-- 用户登录页 -->
    <div id="loginPage" class="page">
        <h2>用户身份验证</h2>
        <div class="form-item">
            <label>姓名</label>
            <input type="text" id="userName" placeholder="请输入您的姓名">
        </div>
        <div class="form-item">
            <label>身份证号</label>
            <input type="text" id="userIdCard" placeholder="请输入18位身份证号" maxlength="18">
        </div>
        <div class="btn-group">
            <button class="btn-default" onclick="go('homePage')">返回首页</button>
            <button class="btn-primary" id="userLoginBtn" onclick="userLogin()">下一步</button>
        </div>
    </div>

    <!-- 缴费页面 -->
    <div id="payPage" class="page">
        <h2>延时费缴纳</h2>
        <div class="amount-box">
            <div class="amount-title">待缴纳延时费</div>
            <div class="amount-num">¥<span id="payAmount">0.00</span></div>
        </div>
        <div class="pay-info">
            <p>缴费人：<span id="showUserName"></span></p>
            <p>身份证号：<span id="showUserIdCard"></span></p>
            <p>收款方：<span id="showReceiveName">白日曛(**桦)</span></p>
        </div>
        <div class="btn-group">
            <button class="btn-default" onclick="userLogout()">退出登录</button>
            <button class="btn-success" id="payBtn" onclick="openPayModal()">微信支付</button>
        </div>
    </div>

    <!-- 管理员登录页 -->
    <div id="adminLoginPage" class="page">
        <h2>管理后台登录</h2>
        <div class="form-item">
            <label>管理员账号</label>
            <input type="text" id="adminAccount" placeholder="请输入管理员账号">
        </div>
        <div class="form-item">
            <label>管理员密码</label>
            <input type="password" id="adminPwd" placeholder="请输入管理员密码">
        </div>
        <div class="btn-group">
            <button class="btn-default" onclick="go('homePage')">返回首页</button>
            <button class="btn-primary" id="adminLoginBtn" onclick="adminLogin()">登录</button>
        </div>
    </div>

    <!-- 管理后台页 -->
    <div id="adminPage" class="page">
        <h2>管理后台</h2>
        
        <h3>收款账户设置</h3>
        <div class="form-item" style="display:flex;gap:10px;">
            <input type="text" id="receiveNameInput" placeholder="微信收款账户名称" style="flex:1;">
            <button class="btn-primary btn-small" id="saveReceiveBtn" onclick="saveReceiveName()">保存收款账户</button>
        </div>

        <h3>用户信息&缴费金额管理 <span id="userCount" class="data-count">共0个用户</span></h3>
        <div style="display:flex;gap:10px;margin-bottom:10px;flex-wrap:wrap;">
            <input type="text" id="addUserName" placeholder="用户姓名" style="flex:1;min-width:100px;">
            <input type="text" id="addUserIdCard" placeholder="用户身份证号" maxlength="18" style="flex:1;min-width:150px;">
            <input type="number" id="addUserAmount" placeholder="应缴金额(元)" style="width:100px;">
        </div>
        <div class="batch-btn-group">
            <button class="btn-primary btn-small" id="addUserBtn" onclick="addUser()">添加/修改用户</button>
            <button class="btn-danger btn-small" id="clearAllUserBtn" onclick="clearAllUser()">一键清空所有用户</button>
        </div>
        <div id="userList" class="user-list">
            <div class="loading">数据加载中...</div>
        </div>

        <h3>缴费订单检索&管理 <span id="orderCount" class="data-count">共0个订单</span></h3>
        <div class="search-box">
            <input type="text" id="searchKey" placeholder="输入姓名或身份证号检索">
            <button class="btn-primary btn-small" onclick="searchOrder()">检索</button>
        </div>
        <div class="batch-btn-group">
            <button class="btn-success btn-small" onclick="exportOrderToExcel()">导出订单Excel</button>
            <button class="btn-danger btn-small" id="clearAllOrderBtn" onclick="clearAllOrder()">一键清空所有订单</button>
        </div>
        <div id="orderList" class="order-list">
            <div class="loading">数据加载中...</div>
        </div>

        <div style="margin-top:20px;">
            <button class="btn-default" onclick="adminLogout()">退出后台</button>
        </div>
    </div>

    <!-- 支付弹窗 -->
    <div id="payModal" class="pay-modal">
        <div class="pay-modal-box">
            <h3 style="margin-top:0;">微信扫码支付</h3>
            <p style="color:#666;margin-bottom:15px;">订单号：<span id="orderNo"></span></p>
            <!-- 引用文档中的微信支付二维码图片 -->
            ![1&-1.0&&&%E5%BE%AE%E4%BF%A1%E6%94%AF%E4%BB%98%E4%BA%8C%E7%BB%B4%E7%A0%81&image&21e772f08cf71a5ab0ae132a7f7c391b&320&&](https://www.coze.cn/s/vwaPZf_VVh4/?a=0)
            <p style="color:#f56c6c;font-size:18px;font-weight:600;">待支付金额：¥<span id="modalAmount">0.00</span></p>
            <p style="color:#999;font-size:12px;margin-top:10px;">请用微信扫码完成付款，管理员确认后自动完成缴费</p>
            <button class="btn-default" onclick="closePayModal()" style="margin-top:20px;">关闭</button>
        </div>
    </div>

    <!-- 提示框 -->
    <div id="toast" class="toast"></div>

    <script>
        // 全局变量
        let currentUser = null;
        let currentOrder = null;
        let app = null;
        let db = null;
        let _ = null;
        let userCollection = null;
        let orderCollection = null;
        let configCollection = null;
        let isLoading = { users: false, orders: false };

        // 腾讯云开发初始化
        async function initCloudBase() {
            try {
                app = window.tcb.init({
                    env: 'cloud-1-2gdzbuhbf53558f0' // 替换为您的环境ID
                });
                
                // 启用匿名登录
                const auth = app.auth({ persistence: 'local' });
                if (!auth.hasLoginState()) {
                    await auth.signInAnonymously();
                }
                
                db = app.database();
                _ = db.command;
                
                // 获取数据库集合引用
                userCollection = db.collection('users');
                orderCollection = db.collection('orders');
                configCollection = db.collection('config');
                
                // 获取默认配置
                await loadConfig();
                console.log('CloudBase 初始化成功');
                
            } catch (error) {
                console.error('CloudBase 初始化失败:', error);
                toast('系统初始化失败，请检查网络或配置');
            }
        }

        // 页面切换
        function go(pageId) {
            document.querySelectorAll('.page').forEach(page => page.classList.remove('active'));
            document.getElementById(pageId).classList.add('active');
        }

        // 全局提示框
        function toast(msg, duration=2000) {
            const toastDom = document.getElementById('toast');
            toastDom.textContent = msg;
            toastDom.classList.add('show');
            setTimeout(() => toastDom.classList.remove('show'), duration);
        }

        // 加载配置（收款账户名）
        async function loadConfig() {
            try {
                const res = await configCollection.where({ _id: 'receiveName' }).get();
                if (res.data.length > 0) {
                    document.getElementById('showReceiveName').textContent = res.data[0].value;
                    document.getElementById('receiveNameInput').value = res.data[0].value;
                } else {
                    // 如果没有配置，创建默认配置
                    await configCollection.add({ _id: 'receiveName', value: '白日曛(**桦)' });
                }
            } catch (error) {
                console.error('加载配置失败:', error);
            }
        }

        // 用户登录验证
        async function userLogin() {
            const name = document.getElementById('userName').value.trim();
            const idCard = document.getElementById('userIdCard').value.trim();
            if(!name || !idCard) return toast('请填写完整的姓名和身份证号');
            if(idCard.length !== 18) return toast('请输入正确的18位身份证号');
            
            const btn = document.getElementById('userLoginBtn');
            btn.disabled = true;
            btn.textContent = '验证中...';
            
            try {
                const res = await userCollection.where({ idCard: idCard }).get();
                if (res.data.length === 0 || res.data[0].name !== name) {
                    toast('身份信息不匹配，请重新输入');
                } else {
                    currentUser = res.data[0];
                    document.getElementById('payAmount').textContent = currentUser.amount;
                    document.getElementById('showUserName').textContent = currentUser.name;
                    document.getElementById('showUserIdCard').textContent = currentUser.idCard;
                    document.getElementById('payBtn').disabled = parseFloat(currentUser.amount) <= 0;
                    go('payPage');
                    toast('身份验证成功');
                }
            } catch (error) {
                console.error('用户登录失败:', error);
                toast('网络错误，请重试');
            } finally {
                btn.disabled = false;
                btn.textContent = '下一步';
            }
        }

        // 打开支付弹窗
        async function openPayModal() {
            if(!currentUser) return toast('用户信息异常，请重新登录');
            if(parseFloat(currentUser.amount) <= 0) return toast('您暂无待缴纳费用');
            
            const btn = document.getElementById('payBtn');
            btn.disabled = true;
            btn.textContent = '生成订单中...';
            
            try {
                // 查询是否有未完成订单
                const res = await orderCollection.where({
                    idCard: currentUser.idCard,
                    status: 'wait'
                }).get();
                
                if (res.data.length > 0) {
                    // 已有未完成订单
                    currentOrder = res.data[0];
                    document.getElementById('orderNo').textContent = currentOrder.orderNo;
                    document.getElementById('modalAmount').textContent = currentOrder.amount;
                    document.getElementById('payModal').style.display = 'flex';
                    toast('已加载您的待支付订单');
                } else {
                    // 创建新订单
                    const orderNo = 'PAY' + Date.now() + Math.floor(Math.random()*1000).toString().padStart(3,'0');
                    const newOrder = {
                        orderNo: orderNo,
                        name: currentUser.name,
                        idCard: currentUser.idCard,
                        amount: currentUser.amount,
                        status: 'wait',
                        createTime: new Date().toLocaleString()
                    };
                    const addRes = await orderCollection.add(newOrder);
                    if (addRes.code) throw new Error(addRes.message);
                    
                    currentOrder = { ...newOrder, _id: addRes.id };
                    document.getElementById('orderNo').textContent = orderNo;
                    document.getElementById('modalAmount').textContent = currentUser.amount;
                    document.getElementById('payModal').style.display = 'flex';
                    toast('支付订单已生成，请扫码付款');
                }
            } catch (error) {
                console.error('生成订单失败:', error);
                toast('订单生成失败，请重试');
            } finally {
                btn.disabled = parseFloat(currentUser.amount) <= 0;
                btn.textContent = '微信支付';
            }
        }

        // 关闭支付弹窗
        function closePayModal() {
            document.getElementById('payModal').style.display = 'none';
            currentOrder = null;
        }

        // 用户退出登录
        function userLogout() {
            currentUser = null;
            currentOrder = null;
            document.getElementById('userName').value = '';
            document.getElementById('userIdCard').value = '';
            go('homePage');
            toast('已退出登录');
        }

        // 管理员登录
        function adminLogin() {
            const account = document.getElementById('adminAccount').value.trim();
            const pwd = document.getElementById('adminPwd').value.trim();
            if(account === 'admin' && pwd === '123456') {
                // 加载配置和列表
                loadConfig();
                renderUserList();
                renderOrderList();
                go('adminPage');
                toast('登录成功');
            } else {
                toast('账号或密码错误');
            }
        }

        // 保存收款账户
        async function saveReceiveName() {
            const name = document.getElementById('receiveNameInput').value.trim();
            if(!name) return toast('请输入收款账户名称');
            
            const btn = document.getElementById('saveReceiveBtn');
            btn.disabled = true;
            btn.textContent = '保存中...';
            
            try {
                const res = await configCollection.where({ _id: 'receiveName' }).get();
                if (res.data.length > 0) {
                    // 更新
                    await configCollection.doc(res.data[0]._id).update({ value: name });
                } else {
                    // 新增
                    await configCollection.add({ _id: 'receiveName', value: name });
                }
                document.getElementById('showReceiveName').textContent = name;
                toast('收款账户保存成功');
            } catch (error) {
                console.error('保存收款账户失败:', error);
                toast('保存失败，请重试');
            } finally {
                btn.disabled = false;
                btn.textContent = '保存收款账户';
            }
        }

        // 渲染用户列表
        async function renderUserList() {
            const dom = document.getElementById('userList');
            const countDom = document.getElementById('userCount');
            
            if (isLoading.users) return;
            isLoading.users = true;
            dom.innerHTML = '<div class="loading">数据加载中...</div>';
            
            try {
                const res = await userCollection.get();
                const userList = res.data || [];
                countDom.textContent = `共${userList.length}个用户`;
                
                if(userList.length === 0) {
                    dom.innerHTML = '<p style="text-align:center;padding:20px;color:#999">暂无用户数据</p>';
                    isLoading.users = false;
                    return;
                }
                
                let html = '';
                userList.forEach(user => {
                    html += `
                    <div class="user-item">
                        <div class="item-content">
                            <div>${user.name}</div>
                            <div style="font-size:12px;color:#999">${user.idCard}</div>
                        </div>
                        <div style="display:flex;align-items:center;gap:10px">
                            <span style="color:#f56c6c;font-weight:600">¥${user.amount}</span>
                            <button class="btn-danger btn-small" onclick="deleteUser('${user._id}', '${user.idCard}')">删除</button>
                        </div>
                    </div>
                    `;
                });
                dom.innerHTML = html;
            } catch (error) {
                console.error('加载用户列表失败:', error);
                dom.innerHTML = '<p style="text-align:center;padding:20px;color:#f56c6c;">加载失败，请刷新重试</p>';
            } finally {
                isLoading.users = false;
            }
        }

        // 添加/修改用户
        async function addUser() {
            const name = document.getElementById('addUserName').value.trim();
            const idCard = document.getElementById('addUserIdCard').value.trim();
            let amount = document.getElementById('addUserAmount').value.trim();
            
            if(!name || !idCard) return toast('请填写完整的用户信息');
            if(idCard.length !== 18) return toast('请输入正确的18位身份证号');
            amount = !amount || isNaN(parseFloat(amount)) ? '0.00' : parseFloat(amount).toFixed(2);
            
            const btn = document.getElementById('addUserBtn');
            btn.disabled = true;
            btn.textContent = '处理中...';
            
            try {
                const res = await userCollection.where({ idCard: idCard }).get();
                
                if(res.data.length > 0) {
                    // 更新用户
                    await userCollection.doc(res.data[0]._id).update({ name: name, amount: amount });
                    toast('用户信息修改成功');
                } else {
                    // 新增用户
                    await userCollection.add({ name, idCard, amount });
                    toast('用户添加成功');
                }
                
                // 清空输入框并刷新列表
                document.getElementById('addUserName').value = '';
                document.getElementById('addUserIdCard').value = '';
                document.getElementById('addUserAmount').value = '';
                renderUserList();
                
            } catch (error) {
                console.error('操作用户失败:', error);
                toast('操作失败，请重试');
            } finally {
                btn.disabled = false;
                btn.textContent = '添加/修改用户';
            }
        }

        // 删除单个用户
        async function deleteUser(userId, idCard) {
            if(!confirm('确定要删除该用户吗？删除后无法恢复')) return;
            
            try {
                // 同时删除该用户的未完成订单
                await orderCollection.where({ idCard: idCard, status: 'wait' }).remove();
                // 删除用户
                await userCollection.doc(userId).remove();
                toast('用户删除成功');
                renderUserList();
                renderOrderList();
            } catch (error) {
                console.error('删除用户失败:', error);
                toast('删除失败，请重试');
            }
        }

        // 一键清空所有用户
        async function clearAllUser() {
            if(!confirm('确定要清空所有用户吗？此操作不可恢复！')) return;
            
            const btn = document.getElementById('clearAllUserBtn');
            btn.disabled = true;
            btn.textContent = '清空中...';
            
            try {
                const res = await userCollection.remove();
                toast('所有用户已清空');
                renderUserList();
            } catch (error) {
                console.error('清空用户失败:', error);
                toast('清空失败，请重试');
            } finally {
                btn.disabled = false;
                btn.textContent = '一键清空所有用户';
            }
        }

        // 渲染订单列表
        async function renderOrderList(searchKey='') {
            const dom = document.getElementById('orderList');
            const countDom = document.getElementById('orderCount');
            
            if (isLoading.orders) return;
            isLoading.orders = true;
            dom.innerHTML = '<div class="loading">数据加载中...</div>';
            
            try {
                let query = orderCollection;
                if (searchKey) {
                    query = query.where({
                        $or: [
                            { name: db.RegExp({ regexp: searchKey, options: 'i' }) },
                            { idCard: db.RegExp({ regexp: searchKey, options: 'i' }) }
                        ]
                    });
                }
                query = query.orderBy('createTime', 'desc');
                
                const res = await query.get();
                const orderList = res.data || [];
                countDom.textContent = `共${orderList.length}个订单`;
                
                if(orderList.length === 0) {
                    dom.innerHTML = '<p style="text-align:center;padding:20px;color:#999">暂无订单数据</p>';
                    isLoading.orders = false;
                    return;
                }
                
                let html = '';
                orderList.forEach(order => {
                    let statusHtml = '';
                    let btnHtml = '';
                    if(order.status === 'wait') {
                        statusHtml = '<span class="status-tag status-wait">待付款</span>';
                        btnHtml = `
                            <div class="item-btn-group">
                                <button class="btn-success btn-small" onclick="confirmPaySuccess('${order._id}', '${order.idCard}', ${order.amount})">确认缴费成功</button>
                                <button class="btn-danger btn-small" onclick="deleteOrder('${order._id}')">删除</button>
                            </div>
                        `;
                    } else {
                        statusHtml = '<span class="status-tag status-success">已完成缴费</span>';
                        btnHtml = `
                            <div class="item-btn-group">
                                <button class="btn-danger btn-small" onclick="deleteOrder('${order._id}')">删除</button>
                            </div>
                        `;
                    }
                    html += `
                    <div class="order-item">
                        <div class="item-content">
                            <div>订单号：${order.orderNo} ${statusHtml}</div>
                            <div style="font-size:12px;color:#666">${order.name} | ${order.idCard} | 金额：¥${order.amount}</div>
                            <div style="font-size:12px;color:#999">创建时间：${order.createTime}</div>
                        </div>
                        ${btnHtml}
                    </div>
                    `;
                });
                dom.innerHTML = html;
            } catch (error) {
                console.error('加载订单列表失败:', error);
                dom.innerHTML = '<p style="text-align:center;padding:20px;color:#f56c6c;">加载失败，请刷新重试</p>';
            } finally {
                isLoading.orders = false;
            }
        }

        // 检索订单
        function searchOrder() {
            const key = document.getElementById('searchKey').value.trim();
            renderOrderList(key);
        }

        // 管理员确认缴费成功
        async function confirmPaySuccess(orderId, userIdCard, amount) {
            if(!confirm('确定已收到该笔款项，确认缴费成功吗？')) return;
            
            try {
                // 1. 更新订单状态
                await orderCollection.doc(orderId).update({ status: 'success' });
                
                // 2. 查找并更新对应的用户金额为0
                const userRes = await userCollection.where({ idCard: userIdCard }).get();
                if (userRes.data.length > 0) {
                    await userCollection.doc(userRes.data[0]._id).update({ amount: '0.00' });
                }
                
                toast('已确认缴费成功，订单已完成');
                renderOrderList(document.getElementById('searchKey').value.trim());
                renderUserList();
            } catch (error) {
                console.error('确认缴费失败:', error);
                toast('操作失败，请重试');
            }
        }

        // 删除单个订单
        async function deleteOrder(orderId) {
            if(!confirm('确定要删除该订单吗？删除后无法恢复')) return;
            
            try {
                await orderCollection.doc(orderId).remove();
                toast('订单删除成功');
                renderOrderList(document.getElementById('searchKey').value.trim());
            } catch (error) {
                console.error('删除订单失败:', error);
                toast('删除失败，请重试');
            }
        }

        // 一键清空所有订单
        async function clearAllOrder() {
            if(!confirm('确定要清空所有订单吗？此操作不可恢复！')) return;
            
            const btn = document.getElementById('clearAllOrderBtn');
            btn.disabled = true;
            btn.textContent = '清空中...';
            
            try {
                const res = await orderCollection.remove();
                toast('所有订单已清空');
                renderOrderList();
            } catch (error) {
                console.error('清空订单失败:', error);
                toast('清空失败，请重试');
            } finally {
                btn.disabled = false;
                btn.textContent = '一键清空所有订单';
            }
        }

        // 一键导出订单为Excel
        async function exportOrderToExcel() {
            try {
                const res = await orderCollection.orderBy('createTime', 'desc').get();
                const orderList = res.data || [];
                
                if(orderList.length === 0) return toast('暂无订单数据，无法导出');
                
                let excelContent = "订单号,姓名,身份证号,缴费金额(元),缴费状态,创建时间\n";
                orderList.forEach(order => {
                    const status = order.status === 'success' ? '已完成缴费' : '待付款';
                    const idCard = `\t${order.idCard}`;
                    excelContent += `${order.orderNo},${order.name},${idCard},${order.amount},${status},${order.createTime}\n`;
                });
                
                const blob = new Blob(['\uFEFF' + excelContent], { type: 'application/vnd.ms-excel;charset=utf-8' });
                const downloadUrl = URL.createObjectURL(blob);
                const aTag = document.createElement('a');
                aTag.href = downloadUrl;
                aTag.download = `延时费缴费订单详情_${new Date().toLocaleDateString().replace(/\//g,'-')}.csv`;
                document.body.appendChild(aTag);
                aTag.click();
                document.body.removeChild(aTag);
                URL.revokeObjectURL(downloadUrl);
                toast('订单Excel导出成功');
                
            } catch (error) {
                console.error('导出订单失败:', error);
                toast('导出失败，请重试');
            }
        }

        // 管理员退出登录
        function adminLogout() {
            document.getElementById('adminAccount').value = '';
            document.getElementById('adminPwd').value = '';
            go('homePage');
            toast('已退出后台');
        }

        // 页面加载初始化
        window.onload = async function() {
            await initCloudBase();
        };
    </script>
</body>
</html>
