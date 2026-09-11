<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Advanced Power Electronics Simulator</title>
    <script src="https://cdn.plot.ly/plotly-2.24.1.min.js"></script>
    <style>
        :root {
            --primary: #2c3e50;
            --accent: #e74c3c;
            --bg: #f8f9fa;
            --panel-bg: #ffffff;
            --border: #e0e0e0;
        }
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg);
            margin: 0;
            padding: 15px;
            color: #333;
        }
        .header {
            text-align: center;
            margin-bottom: 15px;
            color: var(--primary);
        }
        .controls {
            display: flex;
            flex-wrap: wrap;
            gap: 20px;
            justify-content: center;
            align-items: center;
            background: var(--panel-bg);
            padding: 15px;
            border-radius: 8px;
            box-shadow: 0 2px 5px rgba(0,0,0,0.05);
            border: 1px solid var(--border);
            margin-bottom: 20px;
        }
        .control-group {
            display: flex;
            flex-direction: column;
            gap: 5px;
        }
        label {
            font-weight: 600;
            font-size: 14px;
        }
        select, input[type="range"] {
            padding: 8px;
            font-size: 14px;
            border-radius: 4px;
            border: 1px solid #ccc;
            outline: none;
        }
        #alphaValueDisplay {
            font-weight: bold;
            color: var(--accent);
            min-width: 40px;
            display: inline-block;
        }
        .main-layout {
            display: grid;
            grid-template-columns: 350px 1fr;
            gap: 20px;
            align-items: start;
        }
        .diagram-panel {
            background: var(--panel-bg);
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 4px 10px rgba(0,0,0,0.08);
            border: 1px solid var(--border);
            text-align: center;
            position: sticky;
            top: 20px;
        }
        .diagram-panel h3 {
            margin-top: 0;
            color: var(--primary);
            font-size: 16px;
            border-bottom: 2px solid #eee;
            padding-bottom: 10px;
        }
        .plot-panel {
            background: var(--panel-bg);
            padding: 10px;
            border-radius: 8px;
            box-shadow: 0 4px 10px rgba(0,0,0,0.08);
            border: 1px solid var(--border);
        }
        #plot {
            width: 100%;
            height: 750px;
        }
        svg.circuit {
            width: 100%;
            height: 350px;
            display: none;
        }
        svg.circuit.active {
            display: block;
        }
        .note {
            font-size: 12px;
            color: #7f8c8d;
            margin-top: 15px;
            line-height: 1.5;
            text-align: left;
        }
        
        /* Mobile Responsiveness */
        @media (max-width: 900px) {
            .main-layout { grid-template-columns: 1fr; }
            .diagram-panel { position: static; }
        }
    </style>
</head>
<body>

<div class="header">
    <h2>Power Electronics Converter Simulator</h2>
</div>

<div class="controls">
    <div class="control-group">
        <label for="converterType">Select Topology:</label>
        <select id="converterType">
            <option value="1ph-hw">1-Phase Half Wave (R Load)</option>
            <option value="1ph-fw">1-Phase Full Wave (R Load)</option>
            <option value="1ph-semi">1-Phase Semi-Controlled (RL Load + Freewheeling)</option>
            <option value="3ph-hw">3-Phase Half Wave (R Load)</option>
            <option value="3ph-fw">3-Phase Full Wave (RL Load Continuous)</option>
        </select>
    </div>
    
    <div class="control-group">
        <label for="alphaSlider">Firing Angle (α): <span id="alphaValueDisplay">30°</span></label>
        <input type="range" id="alphaSlider" min="0" max="180" value="30" step="1">
    </div>
</div>

<div class="main-layout">
    <!-- Left Side: Circuit Diagrams -->
    <div class="diagram-panel">
        <h3>Circuit Schematic</h3>
        
        <!-- Reusable SVG Components Definitions -->
        <svg style="display:none;">
            <defs>
                <g id="ac-source">
                    <circle cx="20" cy="20" r="15" fill="none" stroke="#2c3e50" stroke-width="2"/>
                    <path d="M 10 20 Q 15 10, 20 20 T 30 20" fill="none" stroke="#2c3e50" stroke-width="1.5"/>
                </g>
                <g id="thyristor">
                    <path d="M 0 15 L 10 15 M 25 15 L 35 15 M 10 5 L 10 25 L 25 15 Z M 25 5 L 25 25 M 18 19 L 14 27" fill="white" stroke="#e74c3c" stroke-width="2"/>
                </g>
                <g id="diode">
                    <path d="M 0 15 L 10 15 M 25 15 L 35 15 M 10 5 L 10 25 L 25 15 Z M 25 5 L 25 25" fill="white" stroke="#3498db" stroke-width="2"/>
                </g>
                <g id="resistor">
                    <path d="M 15 0 L 15 5 L 5 8 L 25 13 L 5 18 L 25 23 L 5 28 L 25 33 L 15 36 L 15 40" fill="none" stroke="#27ae60" stroke-width="2"/>
                </g>
                <g id="inductor">
                    <path d="M 15 0 L 15 5 Q 5 10, 15 15 Q 5 20, 15 25 Q 5 30, 15 35 L 15 40" fill="none" stroke="#f39c12" stroke-width="2"/>
                </g>
            </defs>
        </svg>

        <!-- 1. Single Phase Half Wave -->
        <svg id="svg-1ph-hw" class="circuit active" viewBox="0 0 200 200">
            <use href="#ac-source" x="30" y="80" />
            <use href="#thyristor" x="80" y="45" />
            <use href="#resistor" x="135" y="80" />
            <path d="M 50 60 L 50 60 L 50 60 L 50 60" />
            <!-- Wires -->
            <path d="M 40 80 L 40 60 L 80 60 M 115 60 L 150 60 L 150 80 M 40 115 L 40 140 L 150 140 L 150 120" fill="none" stroke="#333" stroke-width="2"/>
            <!-- Labels -->
            <text x="85" y="45" font-size="12" font-weight="bold">T1</text>
            <text x="165" y="105" font-size="12">Load (R)</text>
            <text x="15" y="125" font-size="12">Vs</text>
        </svg>

        <!-- 2. Single Phase Full Wave -->
        <svg id="svg-1ph-fw" class="circuit" viewBox="0 0 250 200">
            <use href="#ac-source" x="20" y="80" />
            <use href="#thyristor" x="90" y="30" /> <!-- T1 -->
            <use href="#thyristor" x="150" y="30" /> <!-- T3 -->
            <use href="#thyristor" x="90" y="130" /> <!-- T4 -->
            <use href="#thyristor" x="150" y="130" /> <!-- T2 -->
            <use href="#resistor" x="205" y="80" />
            
            <!-- Bridge Wires -->
            <path d="M 40 80 L 40 45 L 90 45 M 40 115 L 40 145 L 90 145 M 125 45 L 150 45 M 125 145 L 150 145 M 185 45 L 220 45 L 220 80 M 185 145 L 220 145 L 220 120 M 105 60 L 105 100 L 165 100 L 165 130" fill="none" stroke="#333" stroke-width="2"/>
            
            <!-- Labels -->
            <text x="100" y="25" font-size="12">T1</text><text x="160" y="25" font-size="12">T3</text>
            <text x="100" y="175" font-size="12">T4</text><text x="160" y="175" font-size="12">T2</text>
            <text x="10" y="125" font-size="12">Vs</text>
        </svg>

        <!-- 3. Single Phase Semi -->
        <svg id="svg-1ph-semi" class="circuit" viewBox="0 0 250 200">
            <use href="#ac-source" x="20" y="80" />
            <use href="#thyristor" x="80" y="20" /> <!-- T1 -->
            <use href="#thyristor" x="140" y="20" /> <!-- T2 -->
            <use href="#diode" x="80" y="140" /> <!-- D1 -->
            <use href="#diode" x="140" y="140" /> <!-- D2 -->
            <!-- Freewheeling -->
            <use href="#diode" x="180" y="70" transform="rotate(90 195 85)" /> 
            
            <use href="#inductor" x="220" y="80" />
            
            <path d="M 40 80 L 40 35 L 80 35 M 40 115 L 40 155 L 80 155 M 115 35 L 140 35 M 115 155 L 140 155 M 175 35 L 235 35 L 235 80 M 175 155 L 235 155 L 235 120 M 205 35 L 205 70 M 205 105 L 205 155" fill="none" stroke="#333" stroke-width="2"/>
            
            <text x="90" y="15" font-size="12">T1</text><text x="150" y="15" font-size="12">T2</text>
            <text x="90" y="185" font-size="12">D1</text><text x="150" y="185" font-size="12">D2</text>
            <text x="165" y="90" font-size="12">FD</text>
        </svg>

        <!-- 4. Three Phase HW -->
        <svg id="svg-3ph-hw" class="circuit" viewBox="0 0 220 200">
            <use href="#ac-source" x="20" y="30" />
            <use href="#ac-source" x="20" y="80" />
            <use href="#ac-source" x="20" y="130" />
            <use href="#thyristor" x="80" y="30" />
            <use href="#thyristor" x="80" y="80" />
            <use href="#thyristor" x="80" y="130" />
            <use href="#resistor" x="175" y="80" />
            
            <path d="M 40 50 L 80 45 M 40 100 L 80 95 M 40 150 L 80 145 M 115 45 L 150 45 L 150 95 L 115 95 M 150 95 L 150 145 L 115 145 M 150 95 L 190 95 L 190 80 M 40 20 L 40 10 L 190 10 L 190 80 M 40 70 L 40 20 M 40 120 L 40 70" fill="none" stroke="#333" stroke-width="2"/>
            
            <text x="0" y="55" font-size="11">Va</text><text x="0" y="105" font-size="11">Vb</text><text x="0" y="155" font-size="11">Vc</text>
            <text x="95" y="25" font-size="11">T1</text><text x="95" y="75" font-size="11">T2</text><text x="95" y="125" font-size="11">T3</text>
        </svg>

        <!-- 5. Three Phase FW -->
        <svg id="svg-3ph-fw" class="circuit" viewBox="0 0 250 200">
            <text x="10" y="45" font-size="11">A</text>
            <text x="10" y="95" font-size="11">B</text>
            <text x="10" y="145" font-size="11">C</text>
            <path d="M 20 40 L 60 40 M 20 90 L 110 90 M 20 140 L 160 140" fill="none" stroke="#333" stroke-width="2"/>
            
            <!-- Upper Group (T1, T3, T5) -->
            <use href="#thyristor" x="50" y="10" transform="rotate(90 65 25)" />
            <use href="#thyristor" x="100" y="10" transform="rotate(90 115 25)" />
            <use href="#thyristor" x="150" y="10" transform="rotate(90 165 25)" />
            <!-- Lower Group (T4, T6, T2) -->
            <use href="#thyristor" x="50" y="140" transform="rotate(90 65 155)" />
            <use href="#thyristor" x="100" y="140" transform="rotate(90 115 155)" />
            <use href="#thyristor" x="150" y="140" transform="rotate(90 165 155)" />
            
            <use href="#inductor" x="215" y="80" />
            
            <!-- Busbars and legs -->
            <path d="M 65 20 L 165 20 L 165 10 M 165 20 L 230 20 L 230 80 M 65 170 L 165 170 L 230 170 L 230 120 M 65 40 L 65 140 M 115 90 L 115 140 M 115 50 L 115 20 M 165 140 L 165 170 M 165 50 L 165 20" fill="none" stroke="#333" stroke-width="2"/>
        </svg>
        
        <div class="note">
            <strong>Key Insights:</strong><br>
            • <strong>Input:</strong> Background waves show the grid voltages the bridge 'sees'.<br>
            • <strong>Gates:</strong> Displays exactly when each thyristor is commanded to fire.<br>
            • <strong>Current:</strong> Highly inductive (RL) loads maintain a constant continuous current, forcing voltage into the negative region in full-wave setups. Pure R loads mirror the voltage curve perfectly.
        </div>
    </div>

    <!-- Right Side: Waveform Plots -->
    <div class="plot-panel">
        <div id="plot"></div>
    </div>
</div>

<script>
    const DEG2RAD = Math.PI / 180;

    const typeSelect = document.getElementById('converterType');
    const alphaSlider = document.getElementById('alphaSlider');
    const alphaDisplay = document.getElementById('alphaValueDisplay');

    typeSelect.addEventListener('change', () => {
        // Toggle SVG Schematic
        document.querySelectorAll('.circuit').forEach(el => el.classList.remove('active'));
        document.getElementById('svg-' + typeSelect.value).classList.add('active');
        updateChart();
    });
    alphaSlider.addEventListener('input', (e) => {
        alphaDisplay.textContent = e.target.value + '°';
        updateChart();
    });

    // 3-Phase Math Helpers
    const Va = (rad) => Math.sin(rad);
    const Vb = (rad) => Math.sin(rad - 120 * DEG2RAD);
    const Vc = (rad) => Math.sin(rad - 240 * DEG2RAD);
    const Vab = (rad) => Va(rad) - Vb(rad);
    const Vac = (rad) => Va(rad) - Vc(rad);
    const Vbc = (rad) => Vb(rad) - Vc(rad);
    const Vba = (rad) => Vb(rad) - Va(rad);
    const Vca = (rad) => Vc(rad) - Va(rad);
    const Vcb = (rad) => Vc(rad) - Vb(rad);

    function triggerPulse(t, triggers, duration = 15) {
        for (let trig of triggers) {
            let modT = t % 360;
            let modTrig = trig % 360;
            // Handle wrap around for pulse
            if (modT >= modTrig && modT <= modTrig + duration) return 1;
        }
        return 0;
    }

    function updateChart() {
        const type = typeSelect.value;
        const alpha = parseInt(alphaSlider.value);
        let t_values = [];
        for (let t = 0; t <= 720; t++) t_values.push(t);

        let traces = [];
        const pulseWidth = 15;

        // Subplot layout config base
        let vOutColor = '#e74c3c';
        let currentMode = 'R'; // R or RL

        if (type === '1ph-hw') {
            let vin=[], gateT1=[], vout=[], iout=[];
            
            for (let t of t_values) {
                let rad = t * DEG2RAD;
                vin.push(Va(rad));
                
                // Gate T1 (alpha, alpha+360)
                let g1 = triggerPulse(t, [alpha, alpha+360], pulseWidth);
                gateT1.push(g1);
                
                let mod = t % 360;
                let v = (mod >= alpha && mod <= 180) ? Va(rad) : 0;
                vout.push(v);
                iout.push(v); // R load
            }
            traces.push({x: t_values, y: vin, name: 'Vin (Va)', yaxis: 'y1', type: 'scatter', line: {color: '#95a5a6'}});
            traces.push({x: t_values, y: gateT1, name: 'T1 Gate', yaxis: 'y2', type: 'scatter', fill: 'tozeroy', line: {color: '#3498db'}});
            traces.push({x: t_values, y: vout, name: 'Vdc', yaxis: 'y3', type: 'scatter', line: {color: vOutColor, width: 3}});
            traces.push({x: t_values, y: iout, name: 'Idc', yaxis: 'y4', type: 'scatter', line: {color: '#27ae60', width: 2}});
        } 
        
        else if (type === '1ph-fw') {
            let vin=[], gateT1T2=[], gateT3T4=[], vout=[], iout=[];
            
            for (let t of t_values) {
                let rad = t * DEG2RAD;
                vin.push(Va(rad));
                
                let g12 = triggerPulse(t, [alpha, alpha+360], pulseWidth);
                let g34 = triggerPulse(t, [alpha+180, alpha+540], pulseWidth);
                gateT1T2.push(g12 ? 2 : 1); // Shift up for visual separation
                gateT3T4.push(g34 ? 1 : 0);
                
                let mod = t % 360;
                let v = 0;
                if (mod >= alpha && mod <= 180) v = Va(rad);
                else if (mod >= 180+alpha && mod <= 360) v = -Va(rad);
                
                vout.push(v);
                iout.push(v); // R Load
            }
            traces.push({x: t_values, y: vin, name: 'Vin (Va)', yaxis: 'y1', type: 'scatter', line: {color: '#bdc3c7', dash: 'dash'}});
            traces.push({x: t_values, y: gateT1T2, name: 'T1,T2', yaxis: 'y2', type: 'scatter', fill: 'tonexty', line: {color: '#3498db'}});
            traces.push({x: t_values, y: gateT3T4, name: 'T3,T4', yaxis: 'y2', type: 'scatter', fill: 'tozeroy', line: {color: '#9b59b6'}});
            traces.push({x: t_values, y: vout, name: 'Vdc', yaxis: 'y3', type: 'scatter', line: {color: vOutColor, width: 3}});
            traces.push({x: t_values, y: iout, name: 'Idc', yaxis: 'y4', type: 'scatter', line: {color: '#27ae60', width: 2}});
        }
        
        else if (type === '1ph-semi') {
            let vin=[], gateT1=[], gateT2=[], vout=[], iout=[];
            
            for (let t of t_values) {
                let rad = t * DEG2RAD;
                vin.push(Va(rad));
                
                let g1 = triggerPulse(t, [alpha, alpha+360], pulseWidth);
                let g2 = triggerPulse(t, [alpha+180, alpha+540], pulseWidth);
                gateT1.push(g1 ? 2 : 1);
                gateT2.push(g2 ? 1 : 0);
                
                let mod = t % 180;
                let v = (mod >= alpha) ? Math.abs(Va(rad)) : 0; // FD cuts negative
                vout.push(v);
                
                // RL Load implies smoothed current
                iout.push(0.6 + 0.1 * Math.sin(2 * rad - Math.PI/4)); // Fake continuous ripple
            }
            traces.push({x: t_values, y: vin, name: 'Vin', yaxis: 'y1', type: 'scatter', line: {color: '#bdc3c7', dash:'dash'}});
            traces.push({x: t_values, y: gateT1, name: 'T1', yaxis: 'y2', type: 'scatter', fill:'tonexty'});
            traces.push({x: t_values, y: gateT2, name: 'T2', yaxis: 'y2', type: 'scatter', fill:'tozeroy'});
            traces.push({x: t_values, y: vout, name: 'Vdc', yaxis: 'y3', type: 'scatter', line: {color: vOutColor, width: 3}});
            traces.push({x: t_values, y: iout, name: 'Idc (RL)', yaxis: 'y4', type: 'scatter', line: {color: '#27ae60', width: 2}});
        }
        
        else if (type === '3ph-hw') {
            let va=[], vb=[], vc=[], gT1=[], gT2=[], gT3=[], vout=[], iout=[];
            let trigs = [30+alpha, 150+alpha, 270+alpha];
            
            for (let t of t_values) {
                let rad = t * DEG2RAD;
                va.push(Va(rad)); vb.push(Vb(rad)); vc.push(Vc(rad));
                
                gT1.push(triggerPulse(t, [trigs[0], trigs[0]+360], pulseWidth) ? 3 : 2);
                gT2.push(triggerPulse(t, [trigs[1], trigs[1]+360], pulseWidth) ? 2 : 1);
                gT3.push(triggerPulse(t, [trigs[2], trigs[2]+360], pulseWidth) ? 1 : 0);
                
                let active = 0;
                for(let i=-1; i<=6; i++) {
                    let trig = 30 + i*120 + alpha;
                    if(t >= trig && t < trig+120) active = ((i%3)+3)%3;
                }
                let val = [Va, Vb, Vc][active](rad);
                vout.push(val > 0 ? val : 0);
                iout.push(val > 0 ? val : 0); // R load
            }
            traces.push({x: t_values, y: va, name: 'Va', yaxis: 'y1', type: 'scatter', line:{color:'#e74c3c', width:1}});
            traces.push({x: t_values, y: vb, name: 'Vb', yaxis: 'y1', type: 'scatter', line:{color:'#f1c40f', width:1}});
            traces.push({x: t_values, y: vc, name: 'Vc', yaxis: 'y1', type: 'scatter', line:{color:'#3498db', width:1}});
            traces.push({x: t_values, y: gT1, name: 'T1', yaxis: 'y2', type: 'scatter', fill:'tonexty'});
            traces.push({x: t_values, y: gT2, name: 'T2', yaxis: 'y2', type: 'scatter', fill:'tonexty'});
            traces.push({x: t_values, y: gT3, name: 'T3', yaxis: 'y2', type: 'scatter', fill:'tozeroy'});
            traces.push({x: t_values, y: vout, name: 'Vdc', yaxis: 'y3', type: 'scatter', line: {color: '#8e44ad', width: 3}});
            traces.push({x: t_values, y: iout, name: 'Idc', yaxis: 'y4', type: 'scatter', line: {color: '#27ae60', width: 2}});
        }
        
        else if (type === '3ph-fw') {
            let vab=[], vac=[], vbc=[], vba=[], vca=[], vcb=[];
            let gates = [[],[],[],[],[],[]];
            let vout=[], iout=[];
            
            // Firing sequence: T1, T2, T3, T4, T5, T6 (60 deg apart)
            let baseTrigs = [60, 120, 180, 240, 300, 360].map(x => x + alpha);
            let funcs = [Vab, Vac, Vbc, Vba, Vca, Vcb];
            
            for (let t of t_values) {
                let rad = t * DEG2RAD;
                vab.push(Vab(rad)); vac.push(Vac(rad)); vbc.push(Vbc(rad));
                vba.push(Vba(rad)); vca.push(Vca(rad)); vcb.push(Vcb(rad));
                
                // Stack 6 gates vertically
                for(let i=0; i<6; i++) {
                    let pulse = triggerPulse(t, [baseTrigs[i]-360, baseTrigs[i], baseTrigs[i]+360], pulseWidth);
                    gates[i].push(pulse ? (6-i) : (5-i));
                }
                
                let active = 0;
                for(let i=-2; i<=15; i++) {
                    let trig = 60*i + alpha;
                    if(t >= trig && t < trig+60) active = ((i%6)+6)%6;
                }
                vout.push(funcs[active](rad));
                iout.push(1.5); // Highly inductive continuous current
            }
            
            let bgStyle = {color: 'rgba(189,195,199,0.3)', width: 1};
            traces.push({x: t_values, y: vab, name: 'Lines', yaxis: 'y1', type: 'scatter', line:bgStyle, showlegend:false});
            traces.push({x: t_values, y: vac, yaxis: 'y1', type: 'scatter', line:bgStyle, showlegend:false});
            traces.push({x: t_values, y: vbc, yaxis: 'y1', type: 'scatter', line:bgStyle, showlegend:false});
            traces.push({x: t_values, y: vba, yaxis: 'y1', type: 'scatter', line:bgStyle, showlegend:false});
            traces.push({x: t_values, y: vca, yaxis: 'y1', type: 'scatter', line:bgStyle, showlegend:false});
            traces.push({x: t_values, y: vcb, yaxis: 'y1', type: 'scatter', line:bgStyle, showlegend:false});
            
            for(let i=0; i<6; i++) {
                traces.push({x: t_values, y: gates[i], name: `T${i+1}`, yaxis: 'y2', type: 'scatter', fill: (i===5)?'tozeroy':'tonexty'});
            }
            traces.push({x: t_values, y: vout, name: 'Vdc', yaxis: 'y3', type: 'scatter', line: {color: '#8e44ad', width: 3}});
            traces.push({x: t_values, y: iout, name: 'Idc (DC)', yaxis: 'y4', type: 'scatter', line: {color: '#27ae60', width: 3}});
        }

        const layout = {
            margin: { l: 60, r: 20, t: 30, b: 40 },
            hovermode: 'x unified',
            showlegend: true,
            legend: { orientation: 'h', y: 1.05, x: 0 },
            xaxis: { 
                title: 'Angle (ωt) in Degrees',
                tickvals: [0, 90, 180, 270, 360, 450, 540, 630, 720],
                range: [0, 720]
            },
            yaxis:  { title: 'AC Input (V)', domain: [0.78, 1], anchor: 'x', fixedrange: true },
            yaxis2: { title: 'Gate Pulses', domain: [0.52, 0.74], anchor: 'x', showticklabels: false, fixedrange: true },
            yaxis3: { title: 'DC Output (V)', domain: [0.26, 0.48], anchor: 'x', fixedrange: true },
            yaxis4: { title: 'Load Current (I)', domain: [0, 0.22], anchor: 'x', fixedrange: true, range: [0, 2] },
        };

        Plotly.newPlot('plot', traces, layout, {responsive: true, displayModeBar: false});
    }

    // Initial render
    updateChart();
</script>

</body>
</html>
