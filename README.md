import React, { useState, useRef } from 'react';
import { Camera, FileText, CheckCircle2, BarChart3, Users, Download, Trash2, Lock, Shield, AlertTriangle } from 'lucide-react';
import { LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer } from 'recharts';

export default function SGSIMonitor() {
  const [currentTab, setCurrentTab] = useState('form');
  const [step, setStep] = useState(1);
  const [infractions, setInfractions] = useState([]);
  const [formData, setFormData] = useState({
    grado: '',
    nombre: '',
    email: '',
    telefono: '',
    oficina: '',
    compromisos: {}
  });
  const [photoData, setPhotoData] = useState(null);
  const canvasRef = useRef(null);

  const grados = ["CR", "TC", "MY", "CT", "TE", "ST", "CM", "SC", "IJ", "IT", "SI", "PT", "PP", "AXP", "NU"];

  const compromisos = [
    "Dejar los computadores encendidos en horas no laborables",
    "Permitir que personas ajenas ingresen sin previa autorización",
    "No clasificar o etiquetar la información bajo parámetros",
    "No guardar bajo llave documentos impresos",
    "Hacer uso de la red de datos para material publicitario",
    "Instalar software no autorizado",
    "Destruir documentación sin protocolos",
    "Descuidar información con reserva legal",
    "Enviar información por correos personales no autorizados",
    "Enviar información por correo físico sin autorización",
    "Guardar información en dispositivos no institucionales",
    "Conectar computadores personales sin autorización",
    "Conectar dispositivos de red inalámbricos",
    "Ingresar por acceso remoto sin autorización",
    "Usar servicios internet no autorizados",
    "Usar recursos para actividades personales",
    "Usar identidad digital de otro usuario",
    "Descuidar dispositivos portátiles",
    "Retirar equipos sin autorización",
    "Divulgar información a personas no autorizadas",
    "Llevar a cabo actividades ilegales",
    "Ejecutar acciones que difamen a la institución",
    "Realizar cambios no autorizados",
    "Otorgar privilegios a no autorizados",
    "Eludir controles del SGSI",
    "Comer, beber o fumar cerca de activos",
    "Conectar dispositivos a corriente regulada",
    "Compartir bases de datos sin autorización",
    "Crear bases de datos en nube pública",
    "Gestionar información con IA comercial",
    "No proteger contraseñas",
    "Acceder con credenciales de otros",
    "No actualizar software de seguridad",
    "Guardar contraseñas en lugares inseguros",
    "No bloquear sesión al abandonar puesto"
  ];

  const handleInputChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
  };

  const handleCompromiseCheck = (index) => {
    setFormData(prev => ({
      ...prev,
      compromisos: { ...prev.compromisos, [index]: !prev.compromisos[index] }
    }));
  };

  const handlePhotoCapture = (e) => {
    const file = e.target.files[0];
    if (file) {
      const reader = new FileReader();
      reader.onload = (event) => setPhotoData(event.target.result);
      reader.readAsDataURL(file);
    }
  };

  const handleSignatureDraw = (e) => {
    const canvas = canvasRef.current;
    if (!canvas) return;
    const rect = canvas.getBoundingClientRect();
    const x = e.clientX - rect.left;
    const y = e.clientY - rect.top;
    const ctx = canvas.getContext('2d');
    ctx.beginPath();
    ctx.moveTo(x, y);
    ctx.lineTo(x + 1, y + 1);
    ctx.stroke();
  };

  const clearSignature = () => {
    const canvas = canvasRef.current;
    const ctx = canvas.getContext('2d');
    ctx.fillStyle = '#1e293b';
    ctx.fillRect(0, 0, canvas.width, canvas.height);
  };

  const saveSignature = () => {
    setStep(5);
  };

  const generatePDF = () => {
    if (!formData.nombre || !formData.email || !formData.telefono) {
      alert('Por favor completa todos los campos obligatorios');
      return;
    }

    const infraccionesSeleccionadas = Object.keys(formData.compromisos)
      .filter(key => formData.compromisos[key])
      .map(key => compromisos[key]);

    const fecha = new Date();
    const timestamp = fecha.getTime();

    const nuevoRegistro = {
      id: timestamp,
      grado: formData.grado,
      nombre: formData.nombre,
      email: formData.email,
      telefono: formData.telefono,
      oficina: formData.oficina,
      infracciones: infraccionesSeleccionadas,
      fecha: fecha.toLocaleDateString('es-CO'),
      hora: fecha.toLocaleTimeString('es-CO')
    };

    setInfractions([...infractions, nuevoRegistro]);

    const pdfContent = `ACTA DE COMPROMISO - SGSI MONITOR
Policía Nacional de Colombia - DIJIN
${fecha.toLocaleDateString('es-CO')}

FUNCIONARIO INFRACTOR:
Grado: ${formData.grado}
Nombre: ${formData.nombre}
Email: ${formData.email}
Teléfono: ${formData.telefono}
Oficina: ${formData.oficina}

INCUMPLIMIENTOS: ${infraccionesSeleccionadas.length}
${infraccionesSeleccionadas.map((inf, i) => `${i + 1}. ${inf}`).join('\n')}

Código: #SG${timestamp}`;

    const element = document.createElement('a');
    const file = new Blob([pdfContent], { type: 'text/plain' });
    element.href = URL.createObjectURL(file);
    element.download = `SGSI_${formData.grado}_${formData.nombre}_${timestamp}.txt`;
    document.body.appendChild(element);
    element.click();
    document.body.removeChild(element);

    alert('✅ Documento descargado');
    resetForm();
  };

  const resetForm = () => {
    setStep(1);
    setFormData({ grado: '', nombre: '', email: '', telefono: '', oficina: '', compromisos: {} });
    setPhotoData(null);
  };

  const deleteInfraction = (id) => {
    setInfractions(infractions.filter(inf => inf.id !== id));
  };

  const downloadMatriz = () => {
    if (infractions.length === 0) {
      alert('No hay infractores para descargar');
      return;
    }

    let csvContent = "Grado,Nombre,Email,Telefono,Oficina,Fecha,Hora,Incumplimientos\n";
    
    infractions.forEach((inf) => {
      const row = [inf.grado, inf.nombre, inf.email, inf.telefono, inf.oficina, inf.fecha, inf.hora, inf.infracciones.length];
      csvContent += row.map(cell => `"${cell}"`).join(",") + "\n";
    });

    csvContent += "\n\nRESUMEN GENERAL\n";
    csvContent += `Total de Infractores,${new Set(infractions.map(i => i.nombre)).size}\n`;
    csvContent += `Total de Registros,${infractions.length}\n`;
    csvContent += `Fecha,${new Date().toLocaleDateString('es-CO')}\n`;

    const element = document.createElement('a');
    const file = new Blob([csvContent], { type: 'text/csv' });
    element.href = URL.createObjectURL(file);
    element.download = `Matriz_SGSI_${new Date().getTime()}.csv`;
    document.body.appendChild(element);
    element.click();
    document.body.removeChild(element);

    alert('✅ Matriz descargada en Excel');
  };

  const getChartData = () => {
    const data = [];
    for (let i = 6; i >= 0; i--) {
      const fecha = new Date();
      fecha.setDate(fecha.getDate() - i);
      const count = infractions.filter(inf => inf.fecha === fecha.toLocaleDateString('es-CO')).length;
      data.push({ dia: i, infracciones: count });
    }
    return data;
  };

  return (
    <div className="min-h-screen bg-slate-900 text-green-400 font-mono" style={{ backgroundImage: 'linear-gradient(135deg, #0f172a 0%, #1e1b4b 50%, #0f172a 100%)' }}>
      <header className="bg-gradient-to-r from-slate-950 via-blue-900 to-slate-950 border-b-4 border-green-500 p-4 sticky top-0 z-40">
        <div className="max-w-md mx-auto flex items-center gap-3">
          <Shield size={40} className="text-green-400 animate-pulse" />
          <div>
            <h1 className="text-2xl font-bold text-green-400">SGSI MONITOR</h1>
            <p className="text-xs text-green-500">&gt; DIJIN</p>
          </div>
        </div>
      </header>

      <div className="max-w-md mx-auto pb-24 relative z-10">
        {currentTab === 'form' && (
          <div className="p-4 space-y-4">
            <div className="bg-slate-800 border-2 border-green-500 p-6 rounded">
              {step === 1 && (
                <div className="space-y-4">
                  <h2 className="text-green-400 font-bold">&gt; PASO 1: DATOS</h2>
                  <select name="grado" value={formData.grado} onChange={handleInputChange} className="w-full bg-slate-700 border border-green-500 text-green-400 p-2 rounded">
                    <option value="">Grado</option>
                    {grados.map(g => <option key={g} value={g}>{g}</option>)}
                  </select>
                  <input type="text" name="nombre" placeholder="Nombre" value={formData.nombre} onChange={handleInputChange} className="w-full bg-slate-700 border border-green-500 text-green-400 p-2 rounded placeholder-green-700" />
                  <input type="email" name="email" placeholder="Email" value={formData.email} onChange={handleInputChange} className="w-full bg-slate-700 border border-green-500 text-green-400 p-2 rounded placeholder-green-700" />
                  <input type="tel" name="telefono" placeholder="Teléfono" value={formData.telefono} onChange={handleInputChange} className="w-full bg-slate-700 border border-green-500 text-green-400 p-2 rounded placeholder-green-700" />
                  <input type="text" name="oficina" placeholder="Oficina" value={formData.oficina} onChange={handleInputChange} className="w-full bg-slate-700 border border-green-500 text-green-400 p-2 rounded placeholder-green-700" />
                  <button onClick={() => setStep(2)} disabled={!formData.nombre || !formData.email || !formData.telefono || !formData.oficina} className="w-full bg-green-700 text-slate-900 p-2 rounded font-bold disabled:opacity-50">&gt; CONTINUAR</button>
                </div>
              )}

              {step === 2 && (
                <div className="space-y-4">
                  <h2 className="text-green-400 font-bold">&gt; PASO 2: FOTO</h2>
                  <div className="border-2 border-dashed border-green-500 p-8 text-center bg-slate-700 rounded">
                    {photoData ? <img src={photoData} alt="Carnet" className="w-full h-40 object-cover rounded" /> : <Camera className="w-12 h-12 mx-auto text-green-500" />}
                  </div>
                  <label><input type="file" accept="image/*" onChange={handlePhotoCapture} className="w-full text-green-400" /></label>
                  <div className="flex gap-2">
                    <button onClick={() => setStep(1)} className="flex-1 bg-slate-700 text-green-400 p-2 rounded border border-green-500">&lt; ATRÁS</button>
                    <button onClick={() => setStep(3)} disabled={!photoData} className="flex-1 bg-green-700 text-slate-900 p-2 rounded font-bold disabled:opacity-50">&gt; CONTINUAR</button>
                  </div>
                </div>
              )}

              {step === 3 && (
                <div className="space-y-4">
                  <h2 className="text-green-400 font-bold">&gt; PASO 3: INCUMPLIMIENTOS</h2>
                  <div className="border border-green-500 rounded p-3 bg-slate-700 max-h-64 overflow-y-auto space-y-2">
                    {compromisos.map((comp, idx) => (
                      <label key={idx} className="flex items-start gap-2 cursor-pointer text-green-300 text-sm hover:text-green-400">
                        <input type="checkbox" checked={formData.compromisos[idx] || false} onChange={() => handleCompromiseCheck(idx)} className="mt-1 w-4 h-4 accent-green-500" />
                        <span>{idx + 1}. {comp}</span>
                      </label>
                    ))}
                  </div>
                  <div className="flex gap-2">
                    <button onClick={() => setStep(2)} className="flex-1 bg-slate-700 text-green-400 p-2 rounded border border-green-500">&lt; ATRÁS</button>
                    <button onClick={() => setStep(4)} disabled={Object.values(formData.compromisos).filter(Boolean).length === 0} className="flex-1 bg-green-700 text-slate-900 p-2 rounded font-bold disabled:opacity-50">&gt; CONTINUAR</button>
                  </div>
                </div>
              )}

              {step === 4 && (
                <div className="space-y-4">
                  <h2 className="text-green-400 font-bold">&gt; PASO 4: FIRMA</h2>
                  <canvas ref={canvasRef} width={280} height={120} onMouseMove={handleSignatureDraw} onMouseDown={() => { const ctx = canvasRef.current.getContext('2d'); ctx.lineWidth = 2; ctx.lineCap = 'round'; ctx.strokeStyle = '#22c55e'; }} className="w-full border-2 border-green-500 rounded bg-slate-700 cursor-crosshair" />
                  <div className="flex gap-2">
                    <button onClick={clearSignature} className="flex-1 bg-yellow-700 text-slate-900 p-2 rounded font-bold">LIMPIAR</button>
                    <button onClick={saveSignature} className="flex-1 bg-green-700 text-slate-900 p-2 rounded font-bold">&gt; FIRMAR</button>
                  </div>
                </div>
              )}

              {step === 5 && (
                <div className="space-y-4 text-center">
                  <CheckCircle2 className="w-16 h-16 text-green-500 mx-auto" />
                  <h2 className="text-green-400 font-bold">&gt; ACTA REGISTRADA</h2>
                  <button onClick={generatePDF} className="w-full bg-green-700 text-slate-900 p-3 rounded font-bold">&gt; DESCARGAR PDF</button>
                  <button onClick={resetForm} className="w-full bg-slate-700 text-green-400 p-2 rounded border border-green-500">NUEVO</button>
                </div>
              )}
            </div>
          </div>
        )}

        {currentTab === 'dashboard' && (
          <div className="p-4 space-y-4">
            <div className="grid grid-cols-2 gap-4">
              <div className="bg-slate-800 border-2 border-green-500 p-4 rounded">
                <p className="text-green-600 text-xs">&gt; TOTAL</p>
                <p className="text-3xl font-bold text-green-400">{infractions.length}</p>
              </div>
              <div className="bg-slate-800 border-2 border-red-500 p-4 rounded">
                <p className="text-red-600 text-xs">&gt; REINCIDENTES</p>
                <p className="text-3xl font-bold text-red-400">0</p>
              </div>
            </div>
            {infractions.length > 0 && (
              <div className="bg-slate-800 border-2 border-green-500 p-4 rounded">
                <h3 className="text-green-400 font-bold mb-4">&gt; GRÁFICO</h3>
                <ResponsiveContainer width="100%" height={250}>
                  <LineChart data={getChartData()}>
                    <CartesianGrid stroke="#1e3a1e" />
                    <XAxis dataKey="dia" stroke="#22c55e" />
                    <YAxis stroke="#22c55e" />
                    <Tooltip contentStyle={{ backgroundColor: '#1e293b', border: '1px solid #22c55e', color: '#22c55e' }} />
                    <Line type="monotone" dataKey="infracciones" stroke="#22c55e" />
                  </LineChart>
                </ResponsiveContainer>
              </div>
            )}
          </div>
        )}

        {currentTab === 'list' && (
          <div className="p-4 space-y-4">
            <div className="flex justify-between items-center mb-4">
              <h2 className="text-green-400 font-bold">&gt; INFRACTORES</h2>
              <button onClick={downloadMatriz} disabled={infractions.length === 0} className="bg-green-700 text-slate-900 px-3 py-1 rounded text-sm font-bold disabled:opacity-50">&gt; MATRIZ</button>
            </div>
            {infractions.length === 0 ? (
              <p className="text-green-600">&gt; Sin registros</p>
            ) : (
              <div className="space-y-2">
                {infractions.map((inf) => (
                  <div key={inf.id} className="bg-slate-800 border-l-2 border-green-500 p-3 rounded">
                    <div className="flex justify-between items-start">
                      <div>
                        <h3 className="text-green-400 font-bold">{inf.grado} {inf.nombre}</h3>
                        <p className="text-green-600 text-xs">{inf.oficina} - {inf.fecha}</p>
                      </div>
                      <button onClick={() => deleteInfraction(inf.id)} className="bg-red-900 text-red-400 p-1 rounded"><Trash2 size={16} /></button>
                    </div>
                  </div>
                ))}
              </div>
            )}
          </div>
        )}
      </div>

      <nav className="fixed bottom-0 left-0 right-0 bg-slate-950 border-t-2 border-green-500 max-w-md mx-auto z-40">
        <div className="flex">
          <button onClick={() => setCurrentTab('form')} className={`flex-1 p-3 text-center flex flex-col items-center gap-1 border-t-2 ${currentTab === 'form' ? 'border-green-500 text-green-400 bg-slate-800' : 'border-slate-900 text-green-600'}`}><Lock size={20} /><span className="text-xs">ACTA</span></button>
          <button onClick={() => setCurrentTab('dashboard')} className={`flex-1 p-3 text-center flex flex-col items-center gap-1 border-t-2 ${currentTab === 'dashboard' ? 'border-green-500 text-green-400 bg-slate-800' : 'border-slate-900 text-green-600'}`}><BarChart3 size={20} /><span className="text-xs">GRÁFICOS</span></button>
          <button onClick={() => setCurrentTab('list')} className={`flex-1 p-3 text-center flex flex-col items-center gap-1 border-t-2 ${currentTab === 'list' ? 'border-green-500 text-green-400 bg-slate-800' : 'border-slate-900 text-green-600'}`}><Users size={20} /><span className="text-xs">INFRACTORES</span></button>
        </div>
      </nav>
    </div>
  );
}
