<?php
session_start();

// Initialize cart
if (!isset($_SESSION['cart'])) {
    $_SESSION['cart'] = [];
}
if (!isset($_SESSION['transactions'])) {
    $_SESSION['transactions'] = [];
}

// Sample products database
$products = [
    ['id' => 1, 'name' => 'Nasi Goreng Spesial', 'price' => 25000, 'category' => 'Makanan', 'stock' => 50, 'emoji' => '🍳'],
    ['id' => 2, 'name' => 'Mie Ayam Bakso', 'price' => 20000, 'category' => 'Makanan', 'stock' => 40, 'emoji' => '🍜'],
    ['id' => 3, 'name' => 'Soto Ayam', 'price' => 18000, 'category' => 'Makanan', 'stock' => 35, 'emoji' => '🥣'],
    ['id' => 4, 'name' => 'Gado-Gado', 'price' => 15000, 'category' => 'Makanan', 'stock' => 30, 'emoji' => '🥗'],
    ['id' => 5, 'name' => 'Ayam Bakar', 'price' => 35000, 'category' => 'Makanan', 'stock' => 25, 'emoji' => '🍗'],
    ['id' => 6, 'name' => 'Es Teh Manis', 'price' => 5000, 'category' => 'Minuman', 'stock' => 100, 'emoji' => '🧋'],
    ['id' => 7, 'name' => 'Jus Alpukat', 'price' => 15000, 'category' => 'Minuman', 'stock' => 60, 'emoji' => '🥤'],
    ['id' => 8, 'name' => 'Air Mineral', 'price' => 3000, 'category' => 'Minuman', 'stock' => 200, 'emoji' => '💧'],
    ['id' => 9, 'name' => 'Kopi Hitam', 'price' => 8000, 'category' => 'Minuman', 'stock' => 80, 'emoji' => '☕'],
    ['id' => 10, 'name' => 'Kerupuk', 'price' => 2000, 'category' => 'Snack', 'stock' => 150, 'emoji' => '🍘'],
    ['id' => 11, 'name' => 'Pisang Goreng', 'price' => 10000, 'category' => 'Snack', 'stock' => 45, 'emoji' => '🍌'],
    ['id' => 12, 'name' => 'Tempe Mendoan', 'price' => 8000, 'category' => 'Snack', 'stock' => 55, 'emoji' => '🫘'],
];

// Handle AJAX requests
if (isset($_POST['action'])) {
    header('Content-Type: application/json');
    
    if ($_POST['action'] === 'add_to_cart') {
        $productId = (int)$_POST['product_id'];
        $found = false;
        foreach ($products as $p) {
            if ($p['id'] === $productId) {
                $found = true;
                if (isset($_SESSION['cart'][$productId])) {
                    $_SESSION['cart'][$productId]['qty']++;
                } else {
                    $_SESSION['cart'][$productId] = [
                        'id' => $p['id'],
                        'name' => $p['name'],
                        'price' => $p['price'],
                        'emoji' => $p['emoji'],
                        'qty' => 1
                    ];
                }
                break;
            }
        }
        echo json_encode(['success' => true, 'cart' => $_SESSION['cart']]);
        exit;
    }
    
    if ($_POST['action'] === 'remove_from_cart') {
        $productId = (int)$_POST['product_id'];
        if (isset($_SESSION['cart'][$productId])) {
            if ($_SESSION['cart'][$productId]['qty'] > 1) {
                $_SESSION['cart'][$productId]['qty']--;
            } else {
                unset($_SESSION['cart'][$productId]);
            }
        }
        echo json_encode(['success' => true, 'cart' => $_SESSION['cart']]);
        exit;
    }
    
    if ($_POST['action'] === 'delete_item') {
        $productId = (int)$_POST['product_id'];
        unset($_SESSION['cart'][$productId]);
        echo json_encode(['success' => true, 'cart' => $_SESSION['cart']]);
        exit;
    }
    
    if ($_POST['action'] === 'checkout') {
        $payment = (float)$_POST['payment'];
        $total = 0;
        foreach ($_SESSION['cart'] as $item) {
            $total += $item['price'] * $item['qty'];
        }
        $change = $payment - $total;
        if ($change < 0) {
            echo json_encode(['success' => false, 'message' => 'Pembayaran kurang!']);
            exit;
        }
        
        $transaction = [
            'id' => 'TRX' . date('YmdHis'),
            'date' => date('d/m/Y H:i:s'),
            'items' => $_SESSION['cart'],
            'total' => $total,
            'payment' => $payment,
            'change' => $change
        ];
        $_SESSION['transactions'][] = $transaction;
        $_SESSION['cart'] = [];
        
        echo json_encode(['success' => true, 'transaction' => $transaction]);
        exit;
    }
    
    if ($_POST['action'] === 'clear_cart') {
        $_SESSION['cart'] = [];
        echo json_encode(['success' => true]);
        exit;
    }
    
    exit;
}

// Calculate cart total
$cartTotal = 0;
foreach ($_SESSION['cart'] as $item) {
    $cartTotal += $item['price'] * $item['qty'];
}
$cartCount = array_sum(array_column($_SESSION['cart'], 'qty'));
?>
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>WarungPOS — Sistem Kasir Modern</title>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link href="https://fonts.googleapis.com/css2?family=Space+Mono:wght@400;700&family=Syne:wght@400;600;700;800&display=swap" rel="stylesheet">
    <style>
        :root {
            --bg: #0d0d0d;
            --surface: #161616;
            --surface2: #1f1f1f;
            --border: #2a2a2a;
            --accent: #f5c842;
            --accent2: #e8483c;
            --green: #3ddc84;
            --text: #f0f0f0;
            --muted: #888;
            --font-display: 'Syne', sans-serif;
            --font-mono: 'Space Mono', monospace;
        }
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            background: var(--bg);
            color: var(--text);
            font-family: var(--font-display);
            min-height: 100vh;
            overflow-x: hidden;
        }

        /* HEADER */
        header {
            background: var(--surface);
            border-bottom: 1px solid var(--border);
            padding: 0 24px;
            height: 60px;
            display: flex;
            align-items: center;
            justify-content: space-between;
            position: sticky;
            top: 0;
            z-index: 100;
        }
        .logo {
            font-size: 20px;
            font-weight: 800;
            letter-spacing: -0.5px;
        }
        .logo span { color: var(--accent); }
        .header-info {
            font-family: var(--font-mono);
            font-size: 11px;
            color: var(--muted);
        }
        .nav-tabs {
            display: flex;
            gap: 4px;
        }
        .nav-tab {
            padding: 6px 16px;
            border-radius: 6px;
            border: 1px solid transparent;
            cursor: pointer;
            font-family: var(--font-display);
            font-size: 13px;
            font-weight: 600;
            transition: all 0.2s;
            background: transparent;
            color: var(--muted);
        }
        .nav-tab.active {
            background: var(--accent);
            color: #000;
            border-color: var(--accent);
        }
        .nav-tab:hover:not(.active) {
            border-color: var(--border);
            color: var(--text);
        }

        /* LAYOUT */
        .app-layout {
            display: grid;
            grid-template-columns: 1fr 360px;
            height: calc(100vh - 60px);
        }

        /* LEFT PANEL - PRODUCTS */
        .products-panel {
            overflow-y: auto;
            padding: 20px;
        }
        .products-panel::-webkit-scrollbar { width: 4px; }
        .products-panel::-webkit-scrollbar-track { background: transparent; }
        .products-panel::-webkit-scrollbar-thumb { background: var(--border); border-radius: 2px; }

        .search-bar {
            display: flex;
            gap: 10px;
            margin-bottom: 20px;
        }
        .search-input {
            flex: 1;
            background: var(--surface);
            border: 1px solid var(--border);
            border-radius: 8px;
            padding: 10px 16px;
            color: var(--text);
            font-family: var(--font-display);
            font-size: 14px;
            outline: none;
            transition: border-color 0.2s;
        }
        .search-input:focus { border-color: var(--accent); }
        .search-input::placeholder { color: var(--muted); }

        .filter-chips {
            display: flex;
            gap: 8px;
            margin-bottom: 20px;
            flex-wrap: wrap;
        }
        .chip {
            padding: 5px 14px;
            border-radius: 20px;
            border: 1px solid var(--border);
            font-size: 12px;
            font-weight: 600;
            cursor: pointer;
            transition: all 0.2s;
            color: var(--muted);
            background: transparent;
        }
        .chip.active {
            background: var(--accent);
            color: #000;
            border-color: var(--accent);
        }

        .products-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(160px, 1fr));
            gap: 12px;
        }
        .product-card {
            background: var(--surface);
            border: 1px solid var(--border);
            border-radius: 12px;
            padding: 16px;
            cursor: pointer;
            transition: all 0.2s;
            position: relative;
            overflow: hidden;
        }
        .product-card::before {
            content: '';
            position: absolute;
            inset: 0;
            background: var(--accent);
            opacity: 0;
            transition: opacity 0.2s;
        }
        .product-card:hover {
            border-color: var(--accent);
            transform: translateY(-2px);
        }
        .product-card:hover::before { opacity: 0.04; }
        .product-card:active { transform: scale(0.97); }
        .product-emoji { font-size: 32px; margin-bottom: 10px; display: block; }
        .product-name { font-size: 13px; font-weight: 700; margin-bottom: 6px; line-height: 1.3; }
        .product-price {
            font-family: var(--font-mono);
            font-size: 13px;
            color: var(--accent);
            font-weight: 700;
        }
        .product-cat {
            font-size: 10px;
            color: var(--muted);
            margin-top: 4px;
            text-transform: uppercase;
            letter-spacing: 0.5px;
        }
        .product-card.in-cart {
            border-color: var(--green);
        }
        .in-cart-badge {
            position: absolute;
            top: 8px;
            right: 8px;
            background: var(--green);
            color: #000;
            font-size: 10px;
            font-weight: 700;
            font-family: var(--font-mono);
            padding: 2px 6px;
            border-radius: 10px;
        }

        /* RIGHT PANEL - CART */
        .cart-panel {
            background: var(--surface);
            border-left: 1px solid var(--border);
            display: flex;
            flex-direction: column;
            overflow: hidden;
        }
        .cart-header {
            padding: 16px 20px;
            border-bottom: 1px solid var(--border);
            display: flex;
            align-items: center;
            justify-content: space-between;
        }
        .cart-title { font-size: 14px; font-weight: 700; }
        .cart-count {
            background: var(--accent);
            color: #000;
            font-family: var(--font-mono);
            font-size: 11px;
            font-weight: 700;
            padding: 2px 8px;
            border-radius: 10px;
        }
        .btn-clear {
            background: transparent;
            border: 1px solid var(--border);
            color: var(--muted);
            font-size: 11px;
            padding: 4px 10px;
            border-radius: 6px;
            cursor: pointer;
            font-family: var(--font-display);
            transition: all 0.2s;
        }
        .btn-clear:hover { border-color: var(--accent2); color: var(--accent2); }

        .cart-items {
            flex: 1;
            overflow-y: auto;
            padding: 12px;
        }
        .cart-items::-webkit-scrollbar { width: 3px; }
        .cart-items::-webkit-scrollbar-thumb { background: var(--border); }

        .cart-empty {
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100%;
            color: var(--muted);
            gap: 10px;
        }
        .cart-empty-icon { font-size: 48px; opacity: 0.3; }
        .cart-empty-text { font-size: 13px; }

        .cart-item {
            background: var(--surface2);
            border: 1px solid var(--border);
            border-radius: 10px;
            padding: 12px;
            margin-bottom: 8px;
            display: grid;
            grid-template-columns: auto 1fr auto;
            gap: 10px;
            align-items: center;
            animation: slideIn 0.2s ease;
        }
        @keyframes slideIn {
            from { opacity: 0; transform: translateX(10px); }
            to { opacity: 1; transform: translateX(0); }
        }
        .cart-item-emoji { font-size: 24px; }
        .cart-item-name { font-size: 12px; font-weight: 700; margin-bottom: 3px; }
        .cart-item-price { font-family: var(--font-mono); font-size: 11px; color: var(--accent); }
        .cart-item-controls {
            display: flex;
            align-items: center;
            gap: 8px;
        }
        .qty-btn {
            width: 26px;
            height: 26px;
            border-radius: 6px;
            border: 1px solid var(--border);
            background: transparent;
            color: var(--text);
            cursor: pointer;
            font-size: 14px;
            font-weight: 700;
            transition: all 0.15s;
            display: flex;
            align-items: center;
            justify-content: center;
        }
        .qty-btn:hover { background: var(--accent); color: #000; border-color: var(--accent); }
        .qty-display {
            font-family: var(--font-mono);
            font-size: 13px;
            font-weight: 700;
            min-width: 20px;
            text-align: center;
        }
        .delete-btn {
            background: transparent;
            border: none;
            color: var(--muted);
            cursor: pointer;
            font-size: 14px;
            padding: 2px;
            transition: color 0.2s;
        }
        .delete-btn:hover { color: var(--accent2); }

        /* CART SUMMARY */
        .cart-summary {
            padding: 16px 20px;
            border-top: 1px solid var(--border);
            background: var(--bg);
        }
        .summary-row {
            display: flex;
            justify-content: space-between;
            font-size: 12px;
            color: var(--muted);
            margin-bottom: 6px;
            font-family: var(--font-mono);
        }
        .summary-total {
            display: flex;
            justify-content: space-between;
            font-size: 18px;
            font-weight: 800;
            margin: 12px 0;
            padding: 10px 0;
            border-top: 1px solid var(--border);
        }
        .total-amount { color: var(--accent); font-family: var(--font-mono); }

        .payment-section {
            margin-top: 10px;
        }
        .payment-label { font-size: 11px; color: var(--muted); margin-bottom: 6px; text-transform: uppercase; letter-spacing: 0.5px; }
        .payment-input {
            width: 100%;
            background: var(--surface);
            border: 1px solid var(--border);
            border-radius: 8px;
            padding: 10px 14px;
            color: var(--text);
            font-family: var(--font-mono);
            font-size: 16px;
            font-weight: 700;
            outline: none;
            transition: border-color 0.2s;
            margin-bottom: 8px;
        }
        .payment-input:focus { border-color: var(--accent); }

        .quick-pay {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 6px;
            margin-bottom: 12px;
        }
        .quick-pay-btn {
            background: var(--surface2);
            border: 1px solid var(--border);
            color: var(--text);
            font-size: 11px;
            font-family: var(--font-mono);
            padding: 6px 4px;
            border-radius: 6px;
            cursor: pointer;
            transition: all 0.2s;
            font-weight: 700;
        }
        .quick-pay-btn:hover { border-color: var(--accent); color: var(--accent); }

        .btn-checkout {
            width: 100%;
            background: var(--accent);
            color: #000;
            border: none;
            border-radius: 10px;
            padding: 14px;
            font-size: 15px;
            font-weight: 800;
            font-family: var(--font-display);
            cursor: pointer;
            transition: all 0.2s;
            letter-spacing: -0.3px;
        }
        .btn-checkout:hover { background: #ffd45e; transform: translateY(-1px); }
        .btn-checkout:active { transform: scale(0.98); }
        .btn-checkout:disabled { background: var(--border); color: var(--muted); cursor: not-allowed; transform: none; }

        /* AI ASSISTANT */
        #ai-panel {
            display: none;
            padding: 20px;
            height: calc(100vh - 60px);
            overflow-y: auto;
        }
        #ai-panel.active { display: flex; flex-direction: column; }
        .ai-header {
            display: flex;
            align-items: center;
            gap: 12px;
            margin-bottom: 20px;
        }
        .ai-avatar {
            width: 44px;
            height: 44px;
            background: linear-gradient(135deg, var(--accent), #ff8c42);
            border-radius: 12px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 22px;
        }
        .ai-title { font-size: 18px; font-weight: 800; }
        .ai-subtitle { font-size: 12px; color: var(--muted); }

        .ai-suggestions {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 10px;
            margin-bottom: 20px;
        }
        .suggestion-btn {
            background: var(--surface);
            border: 1px solid var(--border);
            border-radius: 10px;
            padding: 12px;
            cursor: pointer;
            text-align: left;
            transition: all 0.2s;
            font-family: var(--font-display);
            color: var(--text);
        }
        .suggestion-btn:hover { border-color: var(--accent); }
        .suggestion-icon { font-size: 20px; margin-bottom: 6px; display: block; }
        .suggestion-text { font-size: 12px; font-weight: 600; line-height: 1.4; }

        .chat-container {
            flex: 1;
            background: var(--surface);
            border: 1px solid var(--border);
            border-radius: 12px;
            display: flex;
            flex-direction: column;
            overflow: hidden;
        }
        .chat-messages {
            flex: 1;
            padding: 16px;
            overflow-y: auto;
            display: flex;
            flex-direction: column;
            gap: 12px;
            min-height: 300px;
            max-height: 400px;
        }
        .chat-messages::-webkit-scrollbar { width: 3px; }
        .chat-messages::-webkit-scrollbar-thumb { background: var(--border); }

        .msg {
            display: flex;
            gap: 10px;
            animation: fadeUp 0.3s ease;
        }
        @keyframes fadeUp {
            from { opacity: 0; transform: translateY(8px); }
            to { opacity: 1; transform: translateY(0); }
        }
        .msg.user { flex-direction: row-reverse; }
        .msg-avatar {
            width: 32px;
            height: 32px;
            border-radius: 8px;
            background: var(--surface2);
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 14px;
            flex-shrink: 0;
        }
        .msg.user .msg-avatar { background: var(--accent); }
        .msg-bubble {
            max-width: 75%;
            background: var(--surface2);
            border-radius: 12px 12px 12px 4px;
            padding: 10px 14px;
            font-size: 13px;
            line-height: 1.5;
        }
        .msg.user .msg-bubble {
            background: var(--accent);
            color: #000;
            border-radius: 12px 12px 4px 12px;
            font-weight: 600;
        }
        .typing-indicator {
            display: flex;
            gap: 4px;
            padding: 14px;
        }
        .dot {
            width: 6px;
            height: 6px;
            background: var(--muted);
            border-radius: 50%;
            animation: bounce 1.2s infinite;
        }
        .dot:nth-child(2) { animation-delay: 0.2s; }
        .dot:nth-child(3) { animation-delay: 0.4s; }
        @keyframes bounce {
            0%, 60%, 100% { transform: translateY(0); }
            30% { transform: translateY(-6px); }
        }

        .chat-input-area {
            padding: 12px;
            border-top: 1px solid var(--border);
            display: flex;
            gap: 8px;
        }
        .chat-input {
            flex: 1;
            background: var(--surface2);
            border: 1px solid var(--border);
            border-radius: 8px;
            padding: 10px 14px;
            color: var(--text);
            font-family: var(--font-display);
            font-size: 13px;
            outline: none;
            transition: border-color 0.2s;
        }
        .chat-input:focus { border-color: var(--accent); }
        .chat-input::placeholder { color: var(--muted); }
        .send-btn {
            background: var(--accent);
            border: none;
            border-radius: 8px;
            width: 40px;
            height: 40px;
            cursor: pointer;
            font-size: 16px;
            transition: all 0.2s;
            display: flex;
            align-items: center;
            justify-content: center;
        }
        .send-btn:hover { background: #ffd45e; }

        /* HISTORY PANEL */
        #history-panel {
            display: none;
            padding: 20px;
            height: calc(100vh - 60px);
            overflow-y: auto;
        }
        #history-panel.active { display: block; }
        .history-title { font-size: 20px; font-weight: 800; margin-bottom: 20px; }
        .transaction-card {
            background: var(--surface);
            border: 1px solid var(--border);
            border-radius: 12px;
            padding: 16px;
            margin-bottom: 12px;
        }
        .trx-header {
            display: flex;
            justify-content: space-between;
            margin-bottom: 10px;
        }
        .trx-id { font-family: var(--font-mono); font-size: 12px; color: var(--accent); font-weight: 700; }
        .trx-date { font-size: 11px; color: var(--muted); font-family: var(--font-mono); }
        .trx-items { font-size: 12px; color: var(--muted); margin-bottom: 8px; }
        .trx-footer {
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        .trx-total { font-family: var(--font-mono); font-size: 15px; font-weight: 700; color: var(--green); }
        .trx-badge {
            background: var(--green);
            color: #000;
            font-size: 10px;
            font-weight: 700;
            padding: 2px 8px;
            border-radius: 10px;
        }

        /* MODAL */
        .modal-overlay {
            position: fixed;
            inset: 0;
            background: rgba(0,0,0,0.8);
            z-index: 200;
            display: none;
            align-items: center;
            justify-content: center;
            animation: fadeIn 0.2s;
        }
        .modal-overlay.active { display: flex; }
        @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
        .modal {
            background: var(--surface);
            border: 1px solid var(--border);
            border-radius: 16px;
            padding: 32px;
            width: 420px;
            text-align: center;
            animation: scaleIn 0.25s ease;
        }
        @keyframes scaleIn {
            from { transform: scale(0.9); opacity: 0; }
            to { transform: scale(1); opacity: 1; }
        }
        .modal-icon { font-size: 48px; margin-bottom: 16px; }
        .modal-title { font-size: 22px; font-weight: 800; margin-bottom: 8px; }
        .modal-sub { font-size: 13px; color: var(--muted); margin-bottom: 20px; }
        .receipt-box {
            background: var(--surface2);
            border: 1px dashed var(--border);
            border-radius: 10px;
            padding: 16px;
            text-align: left;
            margin-bottom: 20px;
            font-family: var(--font-mono);
            font-size: 12px;
        }
        .receipt-row {
            display: flex;
            justify-content: space-between;
            margin-bottom: 4px;
        }
        .receipt-total {
            border-top: 1px dashed var(--border);
            padding-top: 8px;
            margin-top: 8px;
            font-size: 14px;
            font-weight: 700;
            color: var(--accent);
        }
        .change-display {
            font-size: 28px;
            font-weight: 800;
            color: var(--green);
            font-family: var(--font-mono);
            margin: 12px 0;
        }
        .btn-modal {
            background: var(--accent);
            color: #000;
            border: none;
            padding: 12px 32px;
            border-radius: 10px;
            font-size: 14px;
            font-weight: 700;
            font-family: var(--font-display);
            cursor: pointer;
            transition: all 0.2s;
        }
        .btn-modal:hover { background: #ffd45e; }

        .panel-main {
            display: contents;
        }
        #main-panel { display: contents; }
        #main-panel.hidden { display: none; }

        /* Scrollbar for panels */
        #ai-panel::-webkit-scrollbar { width: 4px; }
        #ai-panel::-webkit-scrollbar-thumb { background: var(--border); }
        #history-panel::-webkit-scrollbar { width: 4px; }
        #history-panel::-webkit-scrollbar-thumb { background: var(--border); }
    </style>
</head>
<body>

<header>
    <div class="logo">Warung<span>POS</span></div>
    <div class="nav-tabs">
        <button class="nav-tab active" onclick="switchTab('kasir')">🏪 Kasir</button>
        <button class="nav-tab" onclick="switchTab('ai')">🤖 AI Asisten</button>
        <button class="nav-tab" onclick="switchTab('history')">📋 Riwayat</button>
    </div>
    <div class="header-info" id="clock"></div>
</header>

<div id="main-panel" class="app-layout">
    <!-- LEFT: Products -->
    <div class="products-panel">
        <div class="search-bar">
            <input type="text" class="search-input" placeholder="🔍  Cari produk..." id="searchInput" oninput="filterProducts()">
        </div>
        <div class="filter-chips">
            <button class="chip active" onclick="filterCategory('Semua', this)">Semua</button>
            <button class="chip" onclick="filterCategory('Makanan', this)">🍽️ Makanan</button>
            <button class="chip" onclick="filterCategory('Minuman', this)">🥤 Minuman</button>
            <button class="chip" onclick="filterCategory('Snack', this)">🍿 Snack</button>
        </div>
        <div class="products-grid" id="productsGrid">
            <?php foreach ($products as $p): ?>
            <div class="product-card <?= isset($_SESSION['cart'][$p['id']]) ? 'in-cart' : '' ?>"
                 data-id="<?= $p['id'] ?>"
                 data-name="<?= strtolower($p['name']) ?>"
                 data-category="<?= $p['category'] ?>"
                 onclick="addToCart(<?= $p['id'] ?>)">
                <?php if (isset($_SESSION['cart'][$p['id']])): ?>
                <div class="in-cart-badge">×<?= $_SESSION['cart'][$p['id']]['qty'] ?></div>
                <?php endif; ?>
                <span class="product-emoji"><?= $p['emoji'] ?></span>
                <div class="product-name"><?= htmlspecialchars($p['name']) ?></div>
                <div class="product-price">Rp <?= number_format($p['price'], 0, ',', '.') ?></div>
                <div class="product-cat"><?= $p['category'] ?></div>
            </div>
            <?php endforeach; ?>
        </div>
    </div>

    <!-- RIGHT: Cart -->
    <div class="cart-panel">
        <div class="cart-header">
            <div>
                <div class="cart-title">🛒 Keranjang</div>
            </div>
            <div style="display:flex;align-items:center;gap:8px;">
                <span class="cart-count" id="cartCount"><?= $cartCount ?></span>
                <button class="btn-clear" onclick="clearCart()">Kosongkan</button>
            </div>
        </div>

        <div class="cart-items" id="cartItems">
            <?php if (empty($_SESSION['cart'])): ?>
            <div class="cart-empty">
                <div class="cart-empty-icon">🛒</div>
                <div class="cart-empty-text">Keranjang kosong</div>
            </div>
            <?php else: ?>
                <?php foreach ($_SESSION['cart'] as $item): ?>
                <div class="cart-item" id="cart-item-<?= $item['id'] ?>">
                    <span class="cart-item-emoji"><?= $item['emoji'] ?></span>
                    <div>
                        <div class="cart-item-name"><?= htmlspecialchars($item['name']) ?></div>
                        <div class="cart-item-price">Rp <?= number_format($item['price'], 0, ',', '.') ?></div>
                    </div>
                    <div style="display:flex;flex-direction:column;align-items:flex-end;gap:6px;">
                        <button class="delete-btn" onclick="deleteItem(<?= $item['id'] ?>)">✕</button>
                        <div class="cart-item-controls">
                            <button class="qty-btn" onclick="removeItem(<?= $item['id'] ?>)">−</button>
                            <span class="qty-display"><?= $item['qty'] ?></span>
                            <button class="qty-btn" onclick="addToCart(<?= $item['id'] ?>)">+</button>
                        </div>
                    </div>
                </div>
                <?php endforeach; ?>
            <?php endif; ?>
        </div>

        <div class="cart-summary">
            <div class="summary-row">
                <span>Subtotal</span>
                <span id="subtotalDisplay">Rp <?= number_format($cartTotal, 0, ',', '.') ?></span>
            </div>
            <div class="summary-row">
                <span>Items</span>
                <span id="itemsDisplay"><?= $cartCount ?> item</span>
            </div>
            <div class="summary-total">
                <span>TOTAL</span>
                <span class="total-amount" id="totalDisplay">Rp <?= number_format($cartTotal, 0, ',', '.') ?></span>
            </div>

            <div class="payment-section">
                <div class="payment-label">Pembayaran</div>
                <input type="number" class="payment-input" id="paymentInput" placeholder="0" oninput="updateChange()">
                <div class="quick-pay" id="quickPay">
                    <?php
                    $amounts = [50000, 100000, 150000];
                    foreach ($amounts as $amount):
                    ?>
                    <button class="quick-pay-btn" onclick="setPayment(<?= $amount ?>)">
                        <?= number_format($amount/1000, 0) ?>rb
                    </button>
                    <?php endforeach; ?>
                </div>
                <div class="summary-row" id="changeRow" style="display:none;">
                    <span>Kembalian</span>
                    <span id="changeDisplay" style="color: var(--green); font-weight:700;"></span>
                </div>
                <button class="btn-checkout" id="checkoutBtn" onclick="checkout()" <?= empty($_SESSION['cart']) ? 'disabled' : '' ?>>
                    💳 Bayar Sekarang
                </button>
            </div>
        </div>
    </div>
</div>

<!-- AI Panel -->
<div id="ai-panel">
    <div class="ai-header">
        <div class="ai-avatar">🤖</div>
        <div>
            <div class="ai-title">AI Asisten Kasir</div>
            <div class="ai-subtitle">Tanya apapun tentang bisnis kamu</div>
        </div>
    </div>

    <div class="ai-suggestions">
        <button class="suggestion-btn" onclick="askAI('Analisa penjualan hari ini dan berikan rekomendasi')">
            <span class="suggestion-icon">📊</span>
            <div class="suggestion-text">Analisa penjualan hari ini</div>
        </button>
        <button class="suggestion-btn" onclick="askAI('Produk apa yang paling laku dan mengapa?')">
            <span class="suggestion-icon">🏆</span>
            <div class="suggestion-text">Produk terlaris</div>
        </button>
        <button class="suggestion-btn" onclick="askAI('Berikan strategi promosi untuk meningkatkan penjualan')">
            <span class="suggestion-icon">💡</span>
            <div class="suggestion-text">Strategi promosi</div>
        </button>
        <button class="suggestion-btn" onclick="askAI('Hitung keuntungan bersih dan margin profit hari ini')">
            <span class="suggestion-icon">💰</span>
            <div class="suggestion-text">Hitung keuntungan</div>
        </button>
    </div>

    <div class="chat-container">
        <div class="chat-messages" id="chatMessages">
            <div class="msg">
                <div class="msg-avatar">🤖</div>
                <div class="msg-bubble">
                    Halo! Saya AI Asisten untuk warung kamu 👋<br><br>
                    Saya bisa membantu kamu menganalisa penjualan, memberikan rekomendasi strategi bisnis, menghitung keuntungan, atau menjawab pertanyaan seputar operasional kasir. Ada yang bisa saya bantu?
                </div>
            </div>
        </div>
        <div class="chat-input-area">
            <input type="text" class="chat-input" id="chatInput" placeholder="Tanya sesuatu..." onkeydown="if(event.key==='Enter') sendChat()">
            <button class="send-btn" onclick="sendChat()">➤</button>
        </div>
    </div>
</div>

<!-- History Panel -->
<div id="history-panel">
    <div class="history-title">📋 Riwayat Transaksi</div>
    <?php if (empty($_SESSION['transactions'])): ?>
    <div style="text-align:center; color:var(--muted); padding:60px 0;">
        <div style="font-size:48px;margin-bottom:12px;">📭</div>
        <div>Belum ada transaksi</div>
    </div>
    <?php else: ?>
        <?php foreach (array_reverse($_SESSION['transactions']) as $trx): ?>
        <div class="transaction-card">
            <div class="trx-header">
                <span class="trx-id"><?= $trx['id'] ?></span>
                <span class="trx-date"><?= $trx['date'] ?></span>
            </div>
            <div class="trx-items">
                <?php
                $itemNames = array_map(fn($i) => "{$i['name']} ×{$i['qty']}", $trx['items']);
                echo implode(', ', $itemNames);
                ?>
            </div>
            <div class="trx-footer">
                <span class="trx-total">Rp <?= number_format($trx['total'], 0, ',', '.') ?></span>
                <span class="trx-badge">✓ LUNAS</span>
            </div>
        </div>
        <?php endforeach; ?>
    <?php endif; ?>
</div>

<!-- Checkout Modal -->
<div class="modal-overlay" id="checkoutModal">
    <div class="modal">
        <div class="modal-icon">✅</div>
        <div class="modal-title">Transaksi Berhasil!</div>
        <div class="modal-sub">Terima kasih telah berbelanja</div>
        <div class="receipt-box" id="receiptContent"></div>
        <div style="margin-bottom:16px;">
            <div style="font-size:12px;color:var(--muted);margin-bottom:4px;">KEMBALIAN</div>
            <div class="change-display" id="changeModal"></div>
        </div>
        <button class="btn-modal" onclick="closeModal()">Transaksi Baru 🎉</button>
    </div>
</div>

<script>
// Products data for JS
const products = <?= json_encode($products) ?>;

// Clock
function updateClock() {
    const now = new Date();
    const d = now.toLocaleDateString('id-ID', {weekday:'short', day:'numeric', month:'short'});
    const t = now.toLocaleTimeString('id-ID', {hour:'2-digit', minute:'2-digit', second:'2-digit'});
    document.getElementById('clock').textContent = `${d} • ${t}`;
}
updateClock();
setInterval(updateClock, 1000);

// TAB SWITCHING
function switchTab(tab) {
    document.querySelectorAll('.nav-tab').forEach(t => t.classList.remove('active'));
    event.target.classList.add('active');

    const mainPanel = document.getElementById('main-panel');
    const aiPanel = document.getElementById('ai-panel');
    const historyPanel = document.getElementById('history-panel');

    mainPanel.style.display = 'none';
    aiPanel.classList.remove('active');
    historyPanel.classList.remove('active');

    if (tab === 'kasir') {
        mainPanel.style.display = 'grid';
    } else if (tab === 'ai') {
        aiPanel.classList.add('active');
    } else if (tab === 'history') {
        historyPanel.classList.add('active');
        loadHistory();
    }
}

// PRODUCT FILTERING
let currentCategory = 'Semua';
function filterCategory(cat, el) {
    currentCategory = cat;
    document.querySelectorAll('.chip').forEach(c => c.classList.remove('active'));
    el.classList.add('active');
    filterProducts();
}
function filterProducts() {
    const q = document.getElementById('searchInput').value.toLowerCase();
    document.querySelectorAll('.product-card').forEach(card => {
        const name = card.dataset.name;
        const cat = card.dataset.category;
        const matchSearch = name.includes(q);
        const matchCat = currentCategory === 'Semua' || cat === currentCategory;
        card.style.display = (matchSearch && matchCat) ? 'block' : 'none';
    });
}

// CART OPERATIONS
function addToCart(productId) {
    fetch('', {
        method: 'POST',
        headers: {'Content-Type': 'application/x-www-form-urlencoded'},
        body: `action=add_to_cart&product_id=${productId}`
    }).then(r => r.json()).then(data => {
        if (data.success) updateCart(data.cart);
    });
}
function removeItem(productId) {
    fetch('', {
        method: 'POST',
        headers: {'Content-Type': 'application/x-www-form-urlencoded'},
        body: `action=remove_from_cart&product_id=${productId}`
    }).then(r => r.json()).then(data => {
        if (data.success) updateCart(data.cart);
    });
}
function deleteItem(productId) {
    fetch('', {
        method: 'POST',
        headers: {'Content-Type': 'application/x-www-form-urlencoded'},
        body: `action=delete_item&product_id=${productId}`
    }).then(r => r.json()).then(data => {
        if (data.success) updateCart(data.cart);
    });
}
function clearCart() {
    if (!confirm('Kosongkan keranjang?')) return;
    fetch('', {
        method: 'POST',
        headers: {'Content-Type': 'application/x-www-form-urlencoded'},
        body: 'action=clear_cart'
    }).then(r => r.json()).then(data => {
        if (data.success) updateCart({});
    });
}

function formatRp(n) {
    return 'Rp ' + parseInt(n).toLocaleString('id-ID');
}

function updateCart(cart) {
    // Update product cards
    document.querySelectorAll('.product-card').forEach(card => {
        const id = parseInt(card.dataset.id);
        const inCart = cart[id];
        card.classList.toggle('in-cart', !!inCart);
        let badge = card.querySelector('.in-cart-badge');
        if (inCart) {
            if (!badge) {
                badge = document.createElement('div');
                badge.className = 'in-cart-badge';
                card.prepend(badge);
            }
            badge.textContent = '×' + inCart.qty;
        } else {
            if (badge) badge.remove();
        }
    });

    // Update cart items
    const cartItemsEl = document.getElementById('cartItems');
    const items = Object.values(cart);

    if (items.length === 0) {
        cartItemsEl.innerHTML = '<div class="cart-empty"><div class="cart-empty-icon">🛒</div><div class="cart-empty-text">Keranjang kosong</div></div>';
        document.getElementById('checkoutBtn').disabled = true;
    } else {
        let html = '';
        items.forEach(item => {
            html += `<div class="cart-item" id="cart-item-${item.id}">
                <span class="cart-item-emoji">${item.emoji}</span>
                <div>
                    <div class="cart-item-name">${item.name}</div>
                    <div class="cart-item-price">${formatRp(item.price)}</div>
                </div>
                <div style="display:flex;flex-direction:column;align-items:flex-end;gap:6px;">
                    <button class="delete-btn" onclick="deleteItem(${item.id})">✕</button>
                    <div class="cart-item-controls">
                        <button class="qty-btn" onclick="removeItem(${item.id})">−</button>
                        <span class="qty-display">${item.qty}</span>
                        <button class="qty-btn" onclick="addToCart(${item.id})">+</button>
                    </div>
                </div>
            </div>`;
        });
        cartItemsEl.innerHTML = html;
        document.getElementById('checkoutBtn').disabled = false;
    }

    // Update totals
    const total = items.reduce((s, i) => s + i.price * i.qty, 0);
    const count = items.reduce((s, i) => s + i.qty, 0);
    document.getElementById('subtotalDisplay').textContent = formatRp(total);
    document.getElementById('totalDisplay').textContent = formatRp(total);
    document.getElementById('cartCount').textContent = count;
    document.getElementById('itemsDisplay').textContent = count + ' item';

    updateChange();
    updateQuickPay(total);
}

function updateQuickPay(total) {
    if (!total) return;
    const amounts = [
        Math.ceil(total / 50000) * 50000,
        Math.ceil(total / 100000) * 100000,
        Math.ceil(total / 100000) * 100000 + 100000
    ];
    const btns = document.querySelectorAll('.quick-pay-btn');
    amounts.forEach((amt, i) => {
        if (btns[i]) {
            btns[i].textContent = (amt >= 1000000 ? (amt/1000000).toFixed(1)+'jt' : (amt/1000)+'rb');
            btns[i].onclick = () => setPayment(amt);
        }
    });
}

function setPayment(amount) {
    document.getElementById('paymentInput').value = amount;
    updateChange();
}
function updateChange() {
    const totalEl = document.getElementById('totalDisplay').textContent;
    const total = parseInt(totalEl.replace(/[^0-9]/g, ''));
    const payment = parseInt(document.getElementById('paymentInput').value) || 0;
    const change = payment - total;
    const changeRow = document.getElementById('changeRow');
    const changeDisplay = document.getElementById('changeDisplay');
    if (payment > 0 && total > 0) {
        changeRow.style.display = 'flex';
        if (change >= 0) {
            changeDisplay.textContent = formatRp(change);
            changeDisplay.style.color = 'var(--green)';
        } else {
            changeDisplay.textContent = '-' + formatRp(Math.abs(change));
            changeDisplay.style.color = 'var(--accent2)';
        }
    } else {
        changeRow.style.display = 'none';
    }
}

// CHECKOUT
function checkout() {
    const payment = parseInt(document.getElementById('paymentInput').value);
    if (!payment || payment <= 0) {
        alert('Masukkan jumlah pembayaran!');
        return;
    }
    fetch('', {
        method: 'POST',
        headers: {'Content-Type': 'application/x-www-form-urlencoded'},
        body: `action=checkout&payment=${payment}`
    }).then(r => r.json()).then(data => {
        if (data.success) {
            showReceipt(data.transaction);
            updateCart({});
            document.getElementById('paymentInput').value = '';
            document.getElementById('changeRow').style.display = 'none';
        } else {
            alert(data.message);
        }
    });
}

function showReceipt(trx) {
    let html = `<div style="text-align:center;margin-bottom:10px;font-weight:700;">🧾 STRUK PEMBAYARAN</div>`;
    html += `<div class="receipt-row"><span>${trx.id}</span><span>${trx.date}</span></div>`;
    html += `<hr style="border-color:var(--border);margin:8px 0;">`;
    Object.values(trx.items).forEach(item => {
        html += `<div class="receipt-row"><span>${item.name} ×${item.qty}</span><span>${formatRp(item.price * item.qty)}</span></div>`;
    });
    html += `<div class="receipt-row receipt-total"><span>TOTAL</span><span>${formatRp(trx.total)}</span></div>`;
    html += `<div class="receipt-row"><span>Bayar</span><span>${formatRp(trx.payment)}</span></div>`;

    document.getElementById('receiptContent').innerHTML = html;
    document.getElementById('changeModal').textContent = formatRp(trx.change);
    document.getElementById('checkoutModal').classList.add('active');
}
function closeModal() {
    document.getElementById('checkoutModal').classList.remove('active');
    location.reload();
}

// HISTORY
function loadHistory() {
    // History is rendered server-side, just show panel
}

// AI CHAT
async function sendChat() {
    const input = document.getElementById('chatInput');
    const msg = input.value.trim();
    if (!msg) return;
    input.value = '';
    askAI(msg);
}

async function askAI(message) {
    const chatMessages = document.getElementById('chatMessages');

    // Add user message
    chatMessages.innerHTML += `
        <div class="msg user">
            <div class="msg-avatar">👤</div>
            <div class="msg-bubble">${escapeHtml(message)}</div>
        </div>`;

    // Add typing indicator
    const typingId = 'typing-' + Date.now();
    chatMessages.innerHTML += `
        <div class="msg" id="${typingId}">
            <div class="msg-avatar">🤖</div>
            <div class="msg-bubble">
                <div class="typing-indicator">
                    <div class="dot"></div><div class="dot"></div><div class="dot"></div>
                </div>
            </div>
        </div>`;
    chatMessages.scrollTop = chatMessages.scrollHeight;

    // Build context from session
    const transactionSummary = buildTransactionSummary();

    try {
        const response = await fetch('https://api.anthropic.com/v1/messages', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
                model: 'claude-sonnet-4-20250514',
                max_tokens: 1000,
                system: `Kamu adalah AI Asisten untuk sistem kasir warung makan bernama "WarungPOS". 
Tugasmu adalah membantu pemilik warung dengan analisa bisnis, rekomendasi strategi, dan pertanyaan operasional.

Data warung saat ini:
${transactionSummary}

Daftar produk yang tersedia:
${JSON.stringify(products.map(p => ({nama: p.name, harga: p.price, kategori: p.category})))}

Berikan jawaban yang singkat, padat, dan actionable dalam Bahasa Indonesia. Gunakan emoji secukupnya untuk membuat respons lebih menarik.`,
                messages: [{ role: 'user', content: message }]
            })
        });

        const data = await response.json();
        document.getElementById(typingId).remove();

        const reply = data.content?.[0]?.text || 'Maaf, terjadi kesalahan. Coba lagi ya!';
        chatMessages.innerHTML += `
            <div class="msg">
                <div class="msg-avatar">🤖</div>
                <div class="msg-bubble">${formatAIResponse(reply)}</div>
            </div>`;
        chatMessages.scrollTop = chatMessages.scrollHeight;
    } catch (err) {
        document.getElementById(typingId).remove();
        chatMessages.innerHTML += `
            <div class="msg">
                <div class="msg-avatar">🤖</div>
                <div class="msg-bubble" style="color:var(--accent2)">❌ Gagal terhubung ke AI. Pastikan koneksi internet aktif.</div>
            </div>`;
        chatMessages.scrollTop = chatMessages.scrollHeight;
    }
}

function buildTransactionSummary() {
    // Read from DOM/page data
    const cartCountEl = document.getElementById('cartCount');
    const totalEl = document.getElementById('totalDisplay');
    return `Keranjang saat ini: ${cartCountEl?.textContent || 0} item, Total: ${totalEl?.textContent || 'Rp 0'}`;
}

function formatAIResponse(text) {
    return text
        .replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>')
        .replace(/\n/g, '<br>');
}

function escapeHtml(text) {
    return text.replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;');
}

// Init
document.getElementById('main-panel').style.display = 'grid';
</script>

</body>
</html>
