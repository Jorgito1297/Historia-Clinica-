
historia clinica 
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Formulario Interactivo de Historia Clínica</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/signature_pad@4.0.0/dist/signature_pad.umd.min.js"></script>
    <!-- Chosen Palette: Warm Neutrals with Subtle Accents -->
    <!-- Application Structure Plan: A single-page application with a fixed sidebar for navigation between collapsible sections. This structure allows the user to either follow the linear flow of a consultation or jump directly to a specific section (e.g., vital signs in an emergency). Key interactions include interactive diagrams for anatomy, automated calculators for clinical scores and dosages, and a real-time "Red Flag" system to alert users to critical findings. This design was chosen for its efficiency, clarity, and its ability to support clinical decision-making directly within the documentation workflow. -->
    <!-- Visualization & Content Choices: Content from the source documents is organized into logical, interactive sections. The Calgary-Cambridge model structures the consultation flow. Static checklists are transformed into interactive forms with conditional logic. Complex calculations (Glasgow, APGAR, Parkland, dosages) are automated via JavaScript to reduce errors. Visual aids like the abdominal quadrant map and burn diagrams (HTML/CSS/JS, not SVG) are used to enhance understanding and speed up data entry. The goal is to make the form a dynamic tool rather than a static document. -->
    <!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. -->
    <style>
        body { font-family: 'Inter', sans-serif; }
        .red-flag { color: #ef4444; font-weight: 700; }
        .red-flag-icon::after { content: ' 🚩'; }
        .section-content { display: none; }
        .section-content.active { display: block; }
        .nav-link { transition: all 0.2s ease-in-out; }
        .nav-link.active { background-color: #4f46e5; color: white; }
        .input-group { margin-bottom: 1rem; }
        .input-field { width: 100%; padding: 0.5rem; border: 1px solid #d1d5db; border-radius: 0.375rem; }
        .label { display: block; margin-bottom: 0.25rem; font-weight: 500; color: #374151; }
        .collapsible-header { cursor: pointer; }
        .quadrant { transition: background-color 0.2s; }
        .quadrant:hover { background-color: rgba(79, 70, 229, 0.2); }
        #signature-pad { border: 1px solid #d1d5db; border-radius: 0.375rem; }
        .spinner {
            border: 4px solid rgba(0, 0, 0, 0.1);
            width: 24px;
            height: 24px;
            border-radius: 50%;
            border-left-color: #4f46e5;
            animation: spin 1s ease infinite;
            display: inline-block;
            vertical-align: middle;
            margin-left: 0.5rem;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        .anatomical-figure {
            position: relative;
            width: 100%;
            max-width: 300px;
            aspect-ratio: 0.5; /* Approx aspect ratio of human body */
            background-color: #f0f0f0; /* Placeholder for drawing */
            border: 1px solid #ccc;
            margin: auto;
        }
        .lesion-point {
            position: absolute;
            width: 15px;
            height: 15px;
            background-color: red;
            border-radius: 50%;
            cursor: pointer;
            transform: translate(-50%, -50%);
            border: 2px solid white;
        }
        .auscultation-point {
            position: absolute;
            width: 20px;
            height: 20px;
            background-color: rgba(79, 70, 229, 0.6);
            border-radius: 50%;
            cursor: pointer;
            transform: translate(-50%, -50%);
            border: 2px solid white;
            display: flex;
            align-items: center;
            justify-content: center;
            color: white;
            font-size: 0.75rem;
            font-weight: bold;
        }
        .emoji-selector span {
            cursor: pointer;
            font-size: 1.8rem;
            padding: 0.2rem;
            transition: transform 0.1s ease-in-out;
        }
        .emoji-selector span:hover {
            transform: scale(1.2);
        }
        .emoji-selector span.selected {
            background-color: #e0e7ff;
            border-radius: 0.5rem;
        }
    </style>
</head>
<body class="bg-gray-50 text-gray-800">

    <div class="flex h-screen">
        <!-- Sidebar Navigation -->
        <aside class="w-64 bg-white shadow-md p-4 overflow-y-auto">
            <h1 class="text-xl font-bold text-indigo-600 mb-6">Historia Clínica</h1>
            <nav id="navigation-menu" class="space-y-2">
                <a href="#datos-generales" class="nav-link block px-4 py-2 rounded-lg hover:bg-gray-100">1. Datos Generales</a>
                <a href="#inicio" class="nav-link block px-4 py-2 rounded-lg hover:bg-gray-100">2. Inicio de Sesión</a>
                <a href="#motivo-consulta" class="nav-link block px-4 py-2 rounded-lg hover:bg-gray-100">3. Motivo de Consulta</a>
                <a href="#hea" class="nav-link block px-4 py-2 rounded-lg hover:bg-gray-100">4. Historia Enfermedad Actual</a>
                <a href="#antecedentes" class="nav-link block px-4 py-2 rounded-lg hover:bg-gray-100">5. Antecedentes</a>
                <a href="#revision-sistemas" class="nav-link block px-4 py-2 rounded-lg hover:bg-gray-100">6. Revisión por Sistemas</a>
                <a href="#exploracion-fisica" class="nav-link block px-4 py-2 rounded-lg hover:bg-gray-100">7. Exploración Física</a>
                <a href="#evaluaciones-especificas" class="nav-link block px-4 py-2 rounded-lg hover:bg-gray-100">8. Evaluaciones Específicas</a>
                <a href="#plan-manejo" class="nav-link block px-4 py-2 rounded-lg hover:bg-gray-100">9. Plan de Manejo</a>
                <a href="#registro" class="nav-link block px-4 py-2 rounded-lg hover:bg-gray-100">10. Registro y Anexos</a>
            </nav>
        </aside>

        <!-- Main Content -->
        <main class="flex-1 p-6 lg:p-8 overflow-y-auto">
            <div id="main-content">
                <!-- Sections will be dynamically inserted here -->
            </div>
        </main>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', function () {
            const sections = {
                'datos-generales': {
                    title: '1. Datos Generales del Paciente',
                    content: `
                        <p class="mb-4 text-gray-600">Esta sección recopila la información demográfica y administrativa esencial del paciente, que sirve como base para el resto de la historia clínica.</p>
                        <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
                            <div class="input-group"><label class="label">Nombre Completo</label><input type="text" id="patient-name" class="input-field"></div>
                            <div class="input-group"><label class="label">Número de Historia Clínica</label><input type="text" id="patient-id" class="input-field"></div>
                            <div class="input-group"><label class="label">Fuente de Información</label><select id="info-source" class="input-field"><option>Paciente</option><option>Familiar</option><option>Acompañante</option><option>Expediente previo</option></select></div>
                            <div class="input-group"><label class="label">Sala (Opcional)</label><input type="text" id="room" class="input-field"></div>
                            <div class="input-group"><label class="label">Edad</label><input type="number" id="patient-age" class="input-field"></div>
                            <div class="input-group"><label class="label">Fecha de Nacimiento</label><input type="date" id="dob" class="input-field"></div>
                            <div class="input-group"><label class="label">Sexo</label><select id="patient-sex" class="input-field"><option>Masculino</option><option>Femenino</option><option>Otro</option><option>Prefiero no decir</option></select></div>
                            <div class="input-group"><label class="label">Ocupación</label><input type="text" id="occupation" class="input-field"></div>
                            <div class="input-group"><label class="label">Estado Civil</label><select id="marital-status" class="input-field"><option>Soltero/a</option><option>Casado/a</option><option>Unión libre</option><option>Divorciado/a</option><option>Viudo/a</option></select></div>
                            <div class="input-group col-span-1 md:col-span-2"><label class="label">Dirección</label><input type="text" id="address" class="input-field"></div>
                            <div class="input-group"><label class="label">Teléfono de Contacto</label><input type="tel" id="phone" class="input-field"></div>
                            <div class="input-group"><label class="label">Religión</label><input type="text" id="religion" class="input-field"></div>
                        </div>
                    `
                },
                'inicio': {
                    title: '2. Inicio de la Sesión',
                    content: `
                        <p class="mb-4 text-gray-600">Esta sección sigue las pautas del modelo Calgary-Cambridge para establecer una relación terapéutica efectiva desde el primer momento de la consulta.</p>
                        <div class="space-y-4">
                            <div><label class="label">Observación no verbal</label><textarea id="non-verbal-obs" class="input-field"></textarea></div>
                            <div><label class="label">Emoción inicial del paciente</label><input type="text" id="initial-emotion" class="input-field"></div>
                            <div class="input-group">
                                <label class="label">Escala de Ánimo (0-10) - Emojis</label>
                                <div id="mood-emoji-selector" class="flex justify-around text-4xl emoji-selector">
                                    <span data-value="0">😭</span>
                                    <span data-value="2.5">😔</span>
                                    <span data-value="5" class="selected">😐</span>
                                    <span data-value="7.5">😊</span>
                                    <span data-value="10">😄</span>
                                </div>
                                <input type="hidden" id="mood-scale" value="5">
                            </div>
                            <div>
                                <label class="label">Agenda pactada: Quejas principales</label>
                                <div class="space-y-1">
                                    <div class="flex items-center"><input type="checkbox" id="agenda-observar" class="mr-2"><label for="agenda-observar">Observar</label></div>
                                    <div class="flex items-center"><input type="checkbox" id="agenda-escuchar" class="mr-2"><label for="agenda-escuchar">Escuchar</label></div>
                                    <div class="flex items-center"><input type="checkbox" id="agenda-evaluar" class="mr-2"><label for="agenda-evaluar">Evaluar</label></div>
                                    <div class="flex items-center"><input type="checkbox" id="agenda-tratar" class="mr-2"><label for="agenda-tratar">Tratar las quejas principales</label></div>
                                </div>
                            </div>
                            <div class="mt-4 space-y-2">
                                <label class="label">Checklist de Proceso Inicial</label>
                                <div class="flex items-center"><input type="checkbox" class="mr-2">Saludar con nombre</div>
                                <div class="flex items-center"><input type="checkbox" class="mr-2">Establecer confidencialidad</div>
                                <div class="flex items-center"><input type="checkbox" class="mr-2">Validar emoción inicial ("Parece ansioso, es normal")</div>
                            </div>
                        </div>
                    `
                },
                 'motivo-consulta': {
                    title: '3. Motivo de Consulta y Perspectiva del Paciente (ICE)',
                    content: `
                        <p class="mb-4 text-gray-600">Aquí se explora la razón principal de la visita, profundizando en las ideas, preocupaciones y expectativas del paciente (ICE) para entender su perspectiva única.</p>
                        <div class="space-y-4">
                            <div><label class="label">Síntoma principal / Término semiológico (ej. "cefalea")</label><input type="text" id="main-symptom" class="input-field"></div>
                             <div><label class="label">Cronología exacta del síntoma</label><input type="text" id="symptom-chronology" class="input-field" placeholder="¿Cuándo empezó? ¿Cómo ha evolucionado?"></div>
                            <div class="p-4 bg-indigo-50 rounded-lg mt-4">
                                <h3 class="font-semibold text-lg mb-2 text-indigo-700">Perspectiva del Paciente (ICE)</h3>
                                <div><label class="label">I (Ideas): ¿Qué cree que le está pasando?</label><textarea id="patient-ideas" class="input-field"></textarea></div>
                                <div><label class="label">C (Preocupaciones): ¿Qué es lo que más le preocupa de este síntoma?</label><textarea id="patient-concerns" class="input-field"></textarea></div>
                                <div><label class="label">E (Expectativas): ¿Qué espera lograr con esta consulta hoy?</label><textarea id="patient-expectations" class="input-field"></textarea></div>
                            </div>
                             <div class="p-4 bg-gray-100 rounded-lg">
                                <h3 class="font-semibold text-lg mb-2 text-gray-700">Impacto Funcional y Emocional</h3>
                                <div><label class="label">¿Cómo ha afectado este síntoma su vida diaria, trabajo o relaciones?</label><textarea id="functional-impact" class="input-field"></textarea></div>
                                <div class="mt-4">
                                    <label class="label">Escala de Impacto en la Vida (0-10)</label>
                                    <input type="range" min="0" max="10" value="5" id="life-impact-scale" class="w-full"></div>
                                </div>
                            </div>

                        <!-- Asistente de Preguntas Adicionales por IA -->
                        <div class="bg-indigo-50 p-4 rounded-lg shadow mt-6">
                            <h3 class="font-semibold text-xl text-indigo-700">Asistente de Preguntas Adicionales ✨</h3>
                            <p class="mb-4 text-gray-600">Ingrese un síntoma o hallazgo clave para obtener preguntas de seguimiento sugeridas por IA.</p>
                            <div class="input-group">
                                <label class="label">Síntoma o Hallazgo (para generar preguntas)</label>
                                <textarea id="follow-up-symptom-input" class="input-field" rows="3" placeholder="Ej: Fiebre, dolor abdominal, erupción cutánea"></textarea>
                            </div>
                            <button onclick="generateFollowUpQuestions()" id="generate-followup-btn" class="px-4 py-2 bg-pink-600 text-white rounded-lg hover:bg-pink-700">✨ Generar Preguntas</button>
                            <div id="followup-loading" class="mt-4 flex items-center hidden">
                                <div class="spinner"></div>
                                <span class="ml-2 text-gray-600">Generando preguntas...</span>
                            </div>
                            <div id="ai-followup-output" class="mt-4 p-4 bg-gray-100 rounded-lg border border-gray-200 text-gray-700 whitespace-pre-wrap">
                                <!-- Las preguntas de IA aparecerán aquí -->
                            </div>
                            <p class="text-xs text-gray-500 mt-2">La información generada por IA es para fines informativos y de apoyo, NO sustituye el juicio clínico.</p>
                        </div>
                    `
                },
                'hea': {
                    title: '4. Historia de Enfermedad Actual (HEA)',
                    content: `
                        <p class="mb-4 text-gray-600">Detalle el curso de la enfermedad actual, incluyendo su aparición, características, factores que la afectan y tratamientos previos.</p>
                        <div class="space-y-4">
                            <div><label class="label">Aparición</label><select id="hea-onset" class="input-field"><option>Súbito</option><option>Progresivo</option><option>Intermitente</option></select></div>
                            <div>
                                <label class="label">Tipo</label>
                                <div class="space-y-1">
                                    <div class="flex items-center"><input type="checkbox" id="type-pulsatil" class="mr-2"><label for="type-pulsatil">Pulsátil</label></div>
                                    <div class="flex items-center"><input type="checkbox" id="type-opresivo" class="mr-2"><label for="type-opresivo">Opresivo</label></div>
                                    <div class="flex items-center"><input type="checkbox" id="type-quemante" class="mr-2"><label for="type-quemante">Quemante</label></div>
                                    <div class="flex items-center"><input type="checkbox" id="type-colico" class="mr-2"><label for="type-colico">Cólico</label></div>
                                    <div class="flex items-center"><input type="checkbox" id="type-sordo" class="mr-2"><label for="type-sordo">Sordo</label></div>
                                    <div class="flex items-center"><input type="checkbox" id="type-electrico" class="mr-2"><label for="type-electrico">Eléctrico</label></div>
                                    <input type="text" id="hea-type-other" class="input-field mt-1" placeholder="Otro tipo...">
                                </div>
                            </div>
                            <div class="input-group">
                                <label class="label">Intensidad (0-10) - Emojis</label>
                                <div id="intensity-emoji-selector" class="flex justify-around text-4xl emoji-selector">
                                    <span data-value="0">😐</span>
                                    <span data-value="2">🙂</span>
                                    <span data-value="4">😊</span>
                                    <span data-value="6">😟</span>
                                    <span data-value="8"> 😩 </span>
                                    <span data-value="10">😖</span>
                                </div>
                                <input type="hidden" id="hea-intensity" value="0">
                            </div>
                            <div><label class="label">Localización</label><input type="text" id="hea-location" class="input-field"></div>
                            <div><label class="label">Irradiaciones</label><input type="text" id="hea-radiation" class="input-field"></div>
                            <div>
                                <label class="label">Evolución</label>
                                <div class="space-y-1">
                                    <div class="flex items-center"><input type="checkbox" id="evolution-estable" class="mr-2"><label for="evolution-estable">Estable</label></div>
                                    <div class="flex items-center"><input type="checkbox" id="evolution-empeora" class="mr-2"><label for="evolution-empeora">Empeora</label></div>
                                    <div class="flex items-center"><input type="checkbox" id="evolution-mejora" class="mr-2"><label for="evolution-mejora">Mejora</label></div>
                                    <div class="flex items-center"><input type="checkbox" id="evolution-intermitente" class="mr-2"><label for="evolution-intermitente">Intermitente</label></div>
                                </div>
                            </div>
                            <div><label class="label">Fenómenos acompañantes</label><textarea id="hea-accompanying" class="input-field"></textarea></div>
                            <div><label class="label">Factores desencadenantes</label><textarea id="hea-triggers" class="input-field"></textarea></div>
                            <div><label class="label">Factores agravantes</label><textarea id="hea-aggravating" class="input-field"></textarea></div>
                            <div><label class="label">Factores atenuantes</label><textarea id="hea-alleviating" class="input-field"></textarea></div>
                            <div><label class="label">Tratamientos previos</label><textarea id="hea-prev-treatments" class="input-field"></textarea></div>
                            <div><label class="label">Conector Mágico</label><textarea id="hea-magic-connector" class="input-field" placeholder="¿Hay algo que NO le haya preguntado y sea importante?"></textarea></div>
                        </div>
                    `
                },
                'antecedentes': {
                    title: '5. Antecedentes',
                    content: `
                        <p class="mb-4 text-gray-600">Esta sección recopila la información demográfica y administrativa esencial del paciente, que sirve como base para el resto de la historia clínica.</p>
                        <h3 class="font-semibold text-lg mb-2 text-indigo-700">Antecedentes Personales Patológicos</h3>
                        <div class="space-y-2">
                            <div><label class="label">Enfermedades crónicas</label><textarea class="input-field"></textarea></div>
                            <div><label class="label">Medicamentos actuales</label><textarea class="input-field"></textarea></div>
                            <div><label class="label">Alergias (medicamentos, alimentos, etc.)</label><textarea class="input-field"></textarea></div>
                            <div><label class="label">Cirugías</label><textarea class="input-field"></textarea></div>
                            <div><label class="label">Hospitalizaciones Previas</label><textarea class="input-field"></textarea></div>
                        </div>
                        <h3 class="font-semibold text-lg mb-2 mt-6 text-indigo-700">Inmunizaciones</h3>
                        <div class="space-y-2">
                             <div><label class="label">Vacuna COVID-19 (Número de dosis, fabricante, fechas)</label><textarea class="input-field"></textarea></div>
                             <div><label class="label">Otras Vacunas Relevantes</label><textarea class="input-field"></textarea></div>
                        </div>
                        <h3 class="font-semibold text-lg mb-2 mt-6 text-indigo-700">Hábitos Tóxicos</h3>
                        <div class="space-y-2">
                             <div class="input-group">
                                <label class="label">Alcohol</label>
                                <select id="habito-alcohol" class="input-field" onchange="toggleHabitDetail('alcohol')">
                                    <option value="no">No</option><option value="si">Sí</option>
                                </select>
                                <div id="habito-alcohol-detail" class="hidden mt-2 space-y-2">
                                    <input type="text" class="input-field" placeholder="Tipo, frecuencia, cantidad">
                                </div>
                             </div>
                             <div class="input-group">
                                <label class="label">Tabaco</label>
                                <select id="habito-tabaco" class="input-field" onchange="toggleHabitDetail('tabaco')">
                                    <option value="no">No</option><option value="si">Sí</option>
                                </select>
                                <div id="habito-tabaco-detail" class="hidden mt-2 space-y-2">
                                    <input type="text" class="input-field" placeholder="Cigarrillos diarios, años fumando">
                                </div>
                             </div>
                             <div class="input-group">
                                <label class="label">Drogas</label>
                                <select id="habito-drogas" class="input-field" onchange="toggleHabitDetail('drogas')">
                                    <option value="no">No</option><option value="si">Sí</option>
                                </select>
                                <div id="habito-drogas-detail" class="hidden mt-2 space-y-2">
                                    <input type="text" class="input-field" placeholder="Tipo, vía, frecuencia">
                                </div>
                             </div>
                        </div>
                         <h3 class="font-semibold text-lg mb-2 mt-6 text-indigo-700">Antecedentes Sexuales, Reproductivos y Ginecológicos</h3>
                        <div class="space-y-2">
                             <div><label class="label">Menarquia (Primera menstruación)</label><input type="number" class="input-field"></div>
                             <div><label class="label">Pubarquia (Aparición del vello pubiano)</label><input type="number" class="input-field"></div>
                             <div><label class="label">Telarquia (Desarrollo de los senos)</label><input type="number" class="input-field"></div>
                             <div><label class="label">FUM (Fecha Última Menstruación)</label><input type="date" class="input-field"></div>
                             <div><label class="label">Métodos Anticonceptivos</label><input type="text" class="input-field"></div>
                             <div><label class="label">G (Gravidez/Embarazos)</label><input type="number" class="input-field"></div>
                             <div><label class="label">P (Paridad/Partos)</label><input type="number" class="input-field"></div>
                             <div><label class="label">A (Abortos)</label><input type="number" class="input-field"></div>
                             <div><label class="label">C (Cesáreas)</label><input type="number" class="input-field"></div>
                             <div><label class="label">Activo Sexualmente</label><select class="input-field"><option>Sí</option><option>No</option></select></div>
                             <div><label class="label">Preferencia Sexual</label><input type="text" class="input-field"></div>
                             <div><label class="label">Enfermedades de Transmisión Sexual (ETS) Previas</label><textarea class="input-field"></textarea></div>
                        </div>
                    `
                },
                'revision-sistemas': {
                    title: '6. Revisión por Sistemas',
                    content: `
                        <p class="mb-4 text-gray-600">Explore de manera sistemática los síntomas del paciente por cada sistema corporal. La interactividad ayuda a visualizar y documentar los hallazgos de forma eficiente.</p>
                        
                        <details class="bg-white p-4 rounded-lg shadow mb-4">
                            <summary class="font-semibold text-xl cursor-pointer text-indigo-700">6.1. Sistema Cardiovascular</summary>
                            <div class="mt-4 space-y-2">
                                <div><label class="label">Disnea (dificultad para respirar)</label><select class="input-field"><option value="no">No</option><option value="si">Sí</option></select></div>
                                <div><label class="label">Palpitaciones</label><select class="input-field"><option value="no">No</option><option value="si">Sí</option></select></div>
                                <div><label class="label">Dolor Precordial</label><select class="input-field"><option value="no">No</option><option value="si">Sí</option></select></div>
                                <div><label class="label">Ortopnea</label><select class="input-field"><option value="no">No</option><option value="si">Sí</option></select></div>
                                <div><label class="label">Claudicación Intermitente</label><select class="input-field"><option value="no">No</option><option value="si">Sí</option></select></div>
                                <div><label class="label">Edema</label><select class="input-field"><option value="no">No</option><option value="si">Sí</option></select></div>
                                <div><label class="label">Otros hallazgos cardiovasculares</label><textarea class="input-field"></textarea></div>
                            </div>
                        </details>

                        <details class="bg-white p-4 rounded-lg shadow mb-4">
                            <summary class="font-semibold text-xl cursor-pointer text-indigo-700">6.2. Sistema Respiratorio</summary>
                            <div class="mt-4 space-y-4">
                                <h4 class="font-semibold mb-2">Síntomas/Hallazgos Respiratorios (Checklist)</h4>
                                <div class="grid grid-cols-1 md:grid-cols-2 gap-2">
                                    <div class="flex items-center"><input type="checkbox" id="resp-tos" class="mr-2"><label for="resp-tos">Tos</label></div>
                                    <div class="flex items-center"><input type="checkbox" id="resp-hemoptisis" class="mr-2"><label for="resp-hemoptisis">Hemoptisis</label></div>
                                    <div class="flex items-center"><input type="checkbox" id="resp-expectoracion" class="mr-2"><label for="resp-expectoracion">Expectoración</label></div>
                                    <div class="flex items-center"><input type="checkbox" id="resp-puntada" class="mr-2"><label for="resp-puntada">Puntada de Costado</label></div>
                                    <div class="flex items-center"><input type="checkbox" id="resp-cianosis" class="mr-2"><label for="resp-cianosis">Cianosis</label></div>
                                    <div class="flex items-center"><input type="checkbox" id="resp-ronquido" class="mr-2"><label for="resp-ronquido">Ronquido</label></div>
                                    <div class="flex items-center"><input type="checkbox" id="resp-apnea" class="mr-2"><label for="resp-apnea">Apnea del Sueño</label></div>
                                </div>

                                <h4 class="font-semibold mb-2 mt-4">Hallazgos a la Exploración (Auscultación/Percusión)</h4>
                                <div class="space-y-3">
                                    <div>
                                        <label class="label">Claridad</label>
                                        <div class="flex flex-wrap gap-x-4">
                                            <div class="flex items-center"><input type="checkbox" id="claros-der" class="mr-2"><label for="claros-der">Pulmón derecho</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="claros-izq" class="mr-2"><label for="claros-izq">Pulmón izquierdo</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="claros-ambos" class="mr-2"><label for="claros-ambos">Ambos campos pulmonares</label></div>
                                        </div>
                                    </div>
                                    <div>
                                        <label class="label">Sibilancias</label>
                                        <div class="flex flex-wrap gap-x-4">
                                            <div class="flex items-center"><input type="checkbox" id="sibilancias-der" class="mr-2"><label for="sibilancias-der">Pulmón derecho</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="sibilancias-izq" class="mr-2"><label for="sibilancias-izq">Pulmón izquierdo</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="sibilancias-ambos" class="mr-2"><label for="sibilancias-ambos">Ambos campos pulmonares</label></div>
                                        </div>
                                    </div>
                                    <div>
                                        <label class="label">Estertores</label>
                                        <div class="flex flex-wrap gap-x-4">
                                            <div class="flex items-center"><input type="checkbox" id="estertores-der" class="mr-2"><label for="estertores-der">Pulmón derecho</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="estertores-izq" class="mr-2"><label for="estertores-izq">Pulmón izquierdo</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="estertores-ambos" class="mr-2"><label for="estertores-ambos">Ambos campos pulmonares</label></div>
                                        </div>
                                    </div>
                                    <div>
                                        <label class="label">Rales</label>
                                        <div class="flex flex-wrap gap-x-4">
                                            <div class="flex items-center"><input type="checkbox" id="rales-der" class="mr-2"><label for="rales-der">Pulmón derecho</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="rales-izq" class="mr-2"><label for="rales-izq">Pulmón izquierdo</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="rales-ambos" class="mr-2"><label for="rales-ambos">Ambos campos pulmonares</label></div>
                                        </div>
                                    </div>
                                    <div>
                                        <label class="label">Matidez</label>
                                        <div class="flex flex-wrap gap-x-4">
                                            <div class="flex items-center"><input type="checkbox" id="matidez-der" class="mr-2"><label for="matidez-der">Pulmón derecho</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="matidez-izq" class="mr-2"><label for="matidez-izq">Pulmón izquierdo</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="matidez-ambos" class="mr-2"><label for="matidez-ambos">Ambos campos pulmonares</label></div>
                                        </div>
                                    </div>
                                    <div>
                                        <label class="label">Hiperresonancia</label>
                                        <div class="flex flex-wrap gap-x-4">
                                            <div class="flex items-center"><input type="checkbox" id="hiperresonancia-der" class="mr-2"><label for="hiperresonancia-der">Pulmón derecho</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="hiperresonancia-izq" class="mr-2"><label for="hiperresonancia-izq">Pulmón izquierdo</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="hiperresonancia-ambos" class="mr-2"><label for="hiperresonancia-ambos">Ambos campos pulmonares</label></div>
                                        </div>
                                    </div>
                                    <div>
                                        <label class="label">Sonidos Ventilatorios</label>
                                        <div class="space-y-1 ml-4">
                                            <div>
                                                <label class="label text-sm font-normal">Disminuidos</label>
                                                <div class="flex flex-wrap gap-x-4">
                                                    <div class="flex items-center"><input type="checkbox" id="sv-disminuidos-der" class="mr-2"><label for="sv-disminuidos-der">Pulmón derecho</label></div>
                                                    <div class="flex items-center"><input type="checkbox" id="sv-disminuidos-izq" class="mr-2"><label for="sv-disminuidos-izq">Pulmón izquierdo</label></div>
                                                    <div class="flex items-center"><input type="checkbox" id="sv-disminuidos-bilateral" class="mr-2"><label for="sv-disminuidos-bilateral">Bilateral</label></div>
                                                </div>
                                            </div>
                                            <div>
                                                <label class="label text-sm font-normal">Aumentados</label>
                                                <div class="flex flex-wrap gap-x-4">
                                                    <div class="flex items-center"><input type="checkbox" id="sv-aumentados-der" class="mr-2"><label for="sv-aumentados-der">Pulmón derecho</label></div>
                                                    <div class="flex items-center"><input type="checkbox" id="sv-aumentados-izq" class="mr-2"><label for="sv-aumentados-izq">Pulmón izquierdo</label></div>
                                                    <div class="flex items-center"><input type="checkbox" id="sv-aumentados-bilateral" class="mr-2"><label for="sv-aumentados-bilateral">Bilateral</label></div>
                                                </div>
                                            </div>
                                            <div>
                                                <label class="label text-sm font-normal">Ausentes</label>
                                                <div class="flex items-center"><input type="checkbox" id="sv-ausentes-der" class="mr-2"><label for="sv-ausentes-der">Pulmón derecho</label></div>
                                                <div class="flex items-center"><input type="checkbox" id="sv-ausentes-izq" class="mr-2"><label for="sv-ausentes-izq">Pulmón izquierdo</label></div>
                                                <div class="flex items-center"><input type="checkbox" id="sv-ausentes-bilateral" class="mr-2"><label for="sv-ausentes-bilateral">Bilateral</label></div>
                                                </div>
                                            </div>
                                        </div>
                                    <div><label class="label">Otros hallazgos respiratorios</label><textarea class="input-field"></textarea></div>
                                </div>
                            </div>
                        </details>

                        <details class="bg-white p-4 rounded-lg shadow mb-4">
                            <summary class="font-semibold text-xl cursor-pointer text-indigo-700">6.3. Sistema Gastrointestinal</summary>
                            <div class="mt-4 space-y-2">
                                <div><label class="label">Emesis (vómitos)</label><input type="text" class="input-field" placeholder="Frecuencia, contenido, relación con alimentos"></div>
                                <div><label class="label">Diarrea</label><input type="text" class="input-field" placeholder="Frecuencia, consistencia, color, moco/sangre"></div>
                                <div><label class="label">Hematemesis</label><input type="text" class="input-field" placeholder="Cantidad, características"></div>
                                <div><label class="label">Dolor Abdominal Agudo</label><input type="text" class="input-field" placeholder="Tipo, intensidad, irradiación"></div>
                                <div><label class="label">Flatulencia</label><input type="text" class="input-field"></div>
                                <div><label class="label">Pirosis (acidez)</label><input type="text" class="input-field"></div>
                                <div><label class="label">Disfagia</label><input type="text" class="input-field" placeholder="Sólidos/Líquidos/Ambos"></div>
                                <div><label class="label">Reflujo</label><input type="text" class="input-field"></div>
                                <div><label class="label">Otros hallazgos gastrointestinales</label><textarea class="input-field"></textarea></div>
                            </div>
                            <div class="mt-4">
                                <h4 class="font-semibold mb-2">Anatomía Topográfica Abdominal (9 Cuadrantes)</h4>
                                <p class="mb-2 text-gray-600">Haga clic en un cuadrante para ver los órganos principales y explicar sus patologías.</p>
                                <div class="grid grid-cols-3 grid-rows-3 w-64 h-64 mx-auto border-2 border-gray-300">
                                    <div id="hipocondrio-derecho" class="quadrant border-r border-b border-gray-300 flex items-center justify-center text-xs" onclick="showOrgansNineQuadrants('hipocondrio-derecho')">HD</div>
                                    <div id="epigastrio" class="quadrant border-r border-b border-gray-300 flex items-center justify-center text-xs" onclick="showOrgansNineQuadrants('epigastrio')">Epigastrio</div>
                                    <div id="hipocondrio-izquierdo" class="quadrant border-b border-gray-300 flex items-center justify-center text-xs" onclick="showOrgansNineQuadrants('hipocondrio-izquierdo')">HI</div>
                                    <div id="flanco-derecho" class="quadrant border-r border-b border-gray-300 flex items-center justify-center text-xs" onclick="showOrgansNineQuadrants('flanco-derecho')">FD</div>
                                    <div id="mesogastrio" class="quadrant border-r border-b border-gray-300 flex items-center justify-center text-xs" onclick="showOrgansNineQuadrants('mesogastrio')">Mesogastrio</div>
                                    <div id="flanco-izquierdo" class="quadrant border-b border-gray-300 flex items-center justify-center text-xs" onclick="showOrgansNineQuadrants('flanco-izquierdo')">FI</div>
                                    <div id="fosa-iliaca-derecha" class="quadrant border-r border-gray-300 flex items-center justify-center text-xs" onclick="showOrgansNineQuadrants('fosa-iliaca-derecha')">FID</div>
                                    <div id="hipogastrio" class="quadrant border-r border-gray-300 flex items-center justify-center text-xs" onclick="showOrgansNineQuadrants('hipogastrio')">Hipogastrio</div>
                                    <div id="fosa-iliaca-izquierda" class="quadrant flex items-center justify-center text-xs" onclick="showOrgansNineQuadrants('fosa-iliaca-izquierda')">FII</div>
                                </div>
                                <div id="nine-quadrant-info" class="mt-4 p-3 bg-indigo-50 rounded-lg font-medium text-indigo-800" style="display:none;">
                                    <p class="mb-2"><strong>Órganos Principales:</strong> <span id="organs-list"></span></p>
                                    <button onclick="explainOrganPathologies()" class="px-3 py-1 bg-purple-600 text-white rounded-lg hover:bg-purple-700 mt-2">Explicar Patologías ✨</button>
                                    <div id="organ-pathology-output" class="mt-2 text-sm bg-gray-100 p-2 rounded hidden"></div>
                                    <div id="organ-pathology-loading" class="mt-2 flex items-center hidden"><div class="spinner"></div><span class="ml-2 text-gray-600">Generando explicación...</span></div>
                                    <p class="text-xs text-gray-500 mt-2">La información generada por IA es para fines informativos y de apoyo, NO sustituye el juicio clínico.</p>
                                </div>
                            </div>
                        </details>

                        <details class="bg-white p-4 rounded-lg shadow mb-4">
                            <summary class="font-semibold text-xl cursor-pointer text-indigo-700">6.4. Sistema Neurológico</summary>
                            <div class="mt-4 space-y-2">
                                <div><label class="label">Cefalea (características)</label><input type="text" class="input-field"></div>
                                <div><label class="label">Convulsiones (tipo, decorticación/descerebración)</label><input type="text" class="input-field"></div>
                                <div><label class="label">Síncope / Síncope postural</label><input type="text" class="input-field"></div>
                                <div>
                                    <label class="label">Alteración de la conciencia</label>
                                    <select id="alteracion-conciencia" class="input-field">
                                        <option value="">Seleccionar</option>
                                        <option value="Alerta">Alerta</option>
                                        <option value="Consciente Orientado">Consciente Orientado</option>
                                        <option value="Letargico">Letárgico</option>
                                        <option value="Inconsciente">Inconsciente</option>
                                    </select>
                                </div>
                                <div><label class="label">Demencia Senil / Alzheimer</label><input type="text" class="input-field"></div>
                                <div><label class="label">Otros hallazgos neurológicos</label><textarea class="input-field"></textarea></div>
                            </div>
                            <div class="mt-4">
                                <h4 class="font-semibold mb-2">Escala de Coma de Glasgow (GCS)</h4>
                                <div class="space-y-3">
                                    <div>
                                        <label class="label">Apertura Ocular (O)</label>
                                        <select id="gcs-ocular" class="input-field" onchange="calculateGCS()">
                                            <option value="4">4 - Espontánea</option>
                                            <option value="3">3 - A la voz</option>
                                            <option value="2">2 - Al dolor</option>
                                            <option value="1">1 - Ninguna</option>
                                        </select>
                                    </div>
                                    <div>
                                        <label class="label">Respuesta Verbal (V)</label>
                                        <select id="gcs-verbal" class="input-field" onchange="calculateGCS()">
                                            <option value="5">5 - Orientado</option>
                                            <option value="4">4 - Confuso</option>
                                            <option value="3">3 - Palabras inapropiadas</option>
                                            <option value="2">2 - Sonidos incomprensibles</option>
                                            <option value="1">1 - Ninguna</option>
                                        </select>
                                    </div>
                                    <div>
                                        <label class="label">Respuesta Motora (M)</label>
                                        <select id="gcs-motor" class="input-field" onchange="calculateGCS()">
                                            <option value="6">6 - Obedece órdenes</option>
                                            <option value="5">5 - Localiza el dolor</option>
                                            <option value="4">4 - Retirada al dolor</option>
                                            <option value="3">3 - Flexión anormal (Decorticación)</option>
                                            <option value="2">2 - Extensión anormal (Descerebración)</option>
                                            <option value="1">1 - Ninguna</option>
                                        </select>
                                    </div>
                                </div>
                                <div class="mt-4 p-3 bg-gray-100 rounded-lg text-center">
                                    <h5 class="font-semibold">Puntuación Total GCS: <span id="gcs-total" class="text-2xl font-bold">15</span></h5>
                                    <p id="gcs-alert" class="mt-1 font-bold"></p>
                                </div>
                            </div>
                        </details>

                        <details class="bg-white p-4 rounded-lg shadow mb-4">
                            <summary class="font-semibold text-xl cursor-pointer text-indigo-700">6.5. Sistema Osteomuscular</summary>
                            <div class="mt-4 space-y-2">
                                <div class="input-group">
                                    <label class="label">Mialgias (dolor muscular)</label>
                                    <select id="osteo-mialgias" class="input-field" onchange="toggleOsteoDetail('mialgias')">
                                        <option value="no">No</option><option value="si">Sí</option>
                                    </select>
                                    <div id="osteo-mialgias-detail" class="hidden mt-2"><input type="text" class="input-field" placeholder="Características de las mialgias"></div>
                                </div>
                                <div class="input-group">
                                    <label class="label">Artralgia (dolor articular)</label>
                                    <select id="osteo-artralgia" class="input-field" onchange="toggleOsteoDetail('artralgia')">
                                        <option value="no">No</option><option value="si">Sí</option>
                                    </select>
                                    <div id="osteo-artralgia-detail" class="hidden mt-2"><input type="text" class="input-field" placeholder="Características de las artralgias"></div>
                                </div>
                                <div class="input-group"><label class="label">Asimetría</label><select class="input-field"><option value="no">No</option><option value="si">Sí</option></select></div>
                                <div class="input-group"><label class="label">Hemiplegia / Hemiparesia</label><select class="input-field"><option value="no">No</option><option value="si">Sí</option></select></div>
                                <div class="input-group"><label class="label">Hipotonía</label><select class="input-field"><option value="no">No</option><option value="si">Sí</option></select></div>
                                <div class="input-group"><label class="label">Ataxia</label><select class="input-field"><option value="no">No</option><option value="si">Sí</option></select></div>
                                <div class="input-group"><label class="label">Apraxia</label><select class="input-field"><option value="no">No</option><option value="si">Sí</option></select></div>
                                <div><label class="label">Otros hallazgos osteomusculares</label><textarea class="input-field"></textarea></div>
                            </div>
                        </details>

                        <details class="bg-white p-4 rounded-lg shadow mb-4">
                            <summary class="font-semibold text-xl cursor-pointer text-indigo-700">6.6. Sistema Tegumentario (Piel y Anexos)</summary>
                            <div class="mt-4 space-y-2">
                                <div>
                                    <label class="label">Características de la piel</label>
                                    <div class="space-y-1">
                                        <div class="flex items-center"><input type="checkbox" id="skin-fria" class="mr-2"><label for="skin-fria">Fría</label></div>
                                        <div class="flex items-center"><input type="checkbox" id="skin-caliente" class="mr-2"><label for="skin-caliente">Caliente</label></div>
                                        <div class="flex items-center"><input type="checkbox" id="skin-diaforesis" class="mr-2"><label for="skin-diaforesis">Diaforesis (sudoración)</label></div>
                                        <div class="flex items-center"><input type="checkbox" id="skin-palida" class="mr-2"><label for="skin-palida">Pálida</label></div>
                                        <div class="flex items-center"><input type="checkbox" id="skin-humeda" class="mr-2"><label for="skin-humeda">Húmeda</label></div>
                                    </div>
                                </div>
                                <div class="input-group">
                                    <label class="label">Hematomas</label>
                                    <select id="hematoma-present" class="input-field" onchange="toggleHematomaDetails()">
                                        <option value="no">No</option><option value="si">Sí</option>
                                    </select>
                                    <div id="hematoma-details" class="hidden mt-2 space-y-2">
                                        <div><label class="label">Localización del Hematoma</label><input type="text" class="input-field"></div>
                                        <div><label class="label">Tamaño del Hematoma</label>
                                            <select class="input-field">
                                                <option>Grande</option><option>Mediano</option><option>Pequeño</option>
                                            </select>
                                        </div>
                                    </div>
                                </div>
                                <div><label class="label">Prurito (picazón)</label><input type="text" class="input-field"></div>
                                <div><label class="label">Manchas (tipo, características)</label><input type="text" class="input-field"></div>
                                <div><label class="label">Otros hallazgos tegumentarios</label><textarea class="input-field"></textarea></div>
                            </div>
                        </details>

                        <details class="bg-white p-4 rounded-lg shadow mb-4">
                            <summary class="font-semibold text-xl cursor-pointer text-indigo-700">6.7. Sistema Genitourinario</summary>
                            <div class="mt-4 space-y-2">
                                <div class="flex items-center"><input type="checkbox" id="gu-poliuria" class="mr-2"><label for="gu-poliuria">Poliuria (Aumento volumen orina)</label></div>
                                <div class="flex items-center"><input type="checkbox" id="gu-oliguria" class="mr-2"><label for="gu-oliguria">Oliguria (Disminución volumen orina)</label></div>
                                <div class="flex items-center"><input type="checkbox" id="gu-anuria" class="mr-2"><label for="gu-anuria">Anuria (Ausencia de orina)</label></div>
                                <div class="flex items-center"><input type="checkbox" id="gu-piuria" class="mr-2"><label for="gu-piuria">Piuria (Pus en orina)</label></div>
                                <div class="flex items-center"><input type="checkbox" id="gu-hematuria" class="mr-2"><label for="gu-hematuria">Hematuria (Sangre en orina)</label></div>
                                <div class="flex items-center"><input type="checkbox" id="gu-disuria" class="mr-2"><label for="gu-disuria">Disuria (Dolor al orinar)</label></div>
                                <div class="flex items-center"><input type="checkbox" id="gu-polaquiuria" class="mr-2"><label for="gu-polaquiuria">Polaquiuria (Micción frecuente)</label></div>
                                <div class="flex items-center"><input type="checkbox" id="gu-nicturia" class="mr-2"><label for="gu-nicturia">Nicturia (Micción nocturna)</label></div>
                                <div class="flex items-center"><input type="checkbox" id="gu-urgencia" class="mr-2"><label for="gu-urgencia">Urgencia miccional</label></div>
                                <div class="flex items-center"><input type="checkbox" id="gu-tenesmo" class="mr-2"><label for="gu-tenesmo">Tenesmo vesical</label></div>
                                <div class="flex items-center"><input type="checkbox" id="gu-incontinencia" class="mr-2"><label for="gu-incontinencia">Incontinencia urinaria</label></div>
                                <div class="flex items-center"><input type="checkbox" id="gu-dolor-flanco" class="mr-2"><label for="gu-dolor-flanco">Dolor en flanco/suprapúbico</label></div>
                                <div class="flex items-center"><input type="checkbox" id="gu-secreciones" class="mr-2"><label for="gu-secreciones">Secreciones genitales</label></div>
                                <div class="flex items-center"><input type="checkbox" id="gu-lesiones" class="mr-2"><label for="gu-lesiones">Lesiones genitales</label></div>
                                <div><label class="label">Otros hallazgos genitourinarios</label><textarea class="input-field"></textarea></div>
                            </div>
                        </details>

                        <details class="bg-white p-4 rounded-lg shadow mb-4">
                            <summary class="font-semibold text-xl cursor-pointer text-indigo-700">6.8. Sistema Endocrino / Metabólico</summary>
                            <div class="mt-4 space-y-2">
                                <div class="input-group">
                                    <label class="label">Hipertiroidismo</label>
                                    <select id="endo-hipertiroidismo" class="input-field" onchange="toggleEndocrineDetail('hipertiroidismo')">
                                        <option value="no">No</option><option value="si">Sí</option>
                                    </select>
                                    <div id="endo-hipertiroidismo-detail" class="hidden mt-2"><input type="text" class="input-field" placeholder="Síntomas asociados (ej. nerviosismo, pérdida de peso)"></div>
                                </div>
                                <div class="input-group">
                                    <label class="label">Hipotiroidismo</label>
                                    <select id="endo-hipotiroidismo" class="input-field" onchange="toggleEndocrineDetail('hipotiroidismo')">
                                        <option value="no">No</option><option value="si">Sí</option>
                                    </select>
                                    <div id="endo-hipotiroidismo-detail" class="hidden mt-2"><input type="text" class="input-field" placeholder="Síntomas asociados (ej. fatiga, ganancia de peso)"></div>
                                </div>
                                <div><label class="label">Hábitos Nutricionales</label><input type="text" class="input-field"></div>
                                <div><label class="label">Pérdida/Ganancia de Peso (cantidad, tiempo)</label><input type="text" class="input-field"></div>
                                <div><label class="label">Otros hallazgos endocrinos/metabólicos</label><textarea class="input-field"></textarea></div>
                            </div>
                        </details>

                        <details class="bg-white p-4 rounded-lg shadow mb-4">
                            <summary class="font-semibold text-xl cursor-pointer text-indigo-700">6.9. Sistema Oftálmico (Ojos)</summary>
                            <div class="mt-4 space-y-2">
                                <div><label class="label">Agudeza Visual</label><input type="text" class="input-field"></div>
                                <div class="input-group">
                                    <label class="label">Fotofobia</label>
                                    <select id="oft-fotofobia" class="input-field">
                                        <option value="no">No</option><option value="si">Sí</option>
                                    </select>
                                </div>
                                <div><label class="label">Visión Borrosa</label><input type="text" class="input-field"></div>
                                <div class="input-group">
                                    <label class="label">Pupilas</label>
                                    <div class="space-y-1">
                                        <div class="flex items-center"><input type="checkbox" id="pupil-miosis" class="mr-2"><label for="pupil-miosis">Miosis (contraídas)</label></div>
                                        <div class="flex items-center"><input type="checkbox" id="pupil-midriasis" class="mr-2"><label for="pupil-midriasis">Midriasis (dilatadas)</label></div>
                                        <div class="flex items-center"><input type="checkbox" id="pupil-anisocoria" class="mr-2"><label for="pupil-anisocoria">Anisocoria (diferente tamaño)</label></div>
                                        <div class="flex items-center"><input type="checkbox" id="pupil-iguales" class="mr-2"><label for="pupil-iguales">Iguales (tamaño)</label></div>
                                        <div class="flex items-center"><input type="checkbox" id="pupil-reactivas" class="mr-2"><label for="pupil-reactivas">Reactividad a la luz</label></div>
                                    </div>
                                    <div><label class="label mt-2">Observaciones de Pupilas</label><textarea class="input-field"></textarea></div>
                                </div>
                                <div><label class="label">Otros hallazgos oftálmicos</label><textarea class="input-field"></textarea></div>
                            </div>
                        </details>

                        <details class="bg-white p-4 rounded-lg shadow mb-4">
                            <summary class="font-semibold text-xl cursor-pointer text-indigo-700">6.10. Sistema Hematológico / Linfático</summary>
                            <div class="mt-4 space-y-2">
                                <div><label class="label">Palidez</label><input type="text" class="input-field"></div>
                                <div class="input-group">
                                    <label class="label">Hemorragia</label>
                                    <select id="hemorragia-present" class="input-field" onchange="toggleHemorrhageDetails()">
                                        <option value="no">No</option><option value="si">Sí</option>
                                    </select>
                                    <div id="hemorragia-details" class="hidden mt-2 space-y-2">
                                        <div><label class="label">Tipo de Hemorragia</label>
                                            <div class="flex items-center"><input type="checkbox" id="hemorragia-venosa" class="mr-2"><label for="hemorragia-venosa">Venosa</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="hemorragia-arterial" class="mr-2"><label for="hemorragia-arterial">Arterial</label></div>
                                        </div>
                                        <textarea class="input-field mt-1" placeholder="Localización, cantidad, características"></textarea>
                                        <h5 class="font-semibold mt-4">Protocolo de Control de Hemorragias (Stop Bleeding)</h5>
                                        <div class="space-y-1">
                                            <div class="flex items-center"><input type="checkbox" id="protocolo-presion-directa" class="mr-2"><label for="protocolo-presion-directa">Presión Directa</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="protocolo-elevacion" class="mr-2"><label for="protocolo-elevacion">Elevación</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="protocolo-puntos-presion" class="mr-2"><label for="protocolo-puntos-presion">Puntos de Presión</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="protocolo-torniquete" class="mr-2"><label for="protocolo-torniquete">Torniquete (si aplica)</label></div>
                                            <div><label class="label mt-2">Medidas Adicionales:</label><textarea class="input-field"></textarea></div>
                                        </div>
                                    </div>
                                </div>
                                <div><label class="label">Petequias</label><input type="text" class="input-field"></div>
                                <div><label class="label">Adenopatías (localización, tamaño)</label><input type="text" class="input-field"></div>
                                <div><label class="label">Otros hallazgos hematológicos/linfáticos</label><textarea class="input-field"></textarea></div>
                            </div>
                        </details>

                        <details class="bg-white p-4 rounded-lg shadow mb-4">
                            <summary class="font-semibold text-xl cursor-pointer text-indigo-700">6.11. Sistema de Evaluación Rápida</summary>
                            <div class="mt-4 space-y-2">
                                <h4 class="font-semibold text-lg">Examinación Primaria (ABCD)</h4>
                                <div class="input-group">
                                    <label class="label">Vía Aérea</label>
                                    <select id="via-aerea-primaria" class="input-field">
                                        <option value="">Seleccionar</option><option value="Patente">Patente</option><option value="No Patente">No Patente</option>
                                    </select>
                                </div>
                                <div class="input-group">
                                    <label class="label">Circulación</label>
                                    <select id="circulacion-primaria" class="input-field">
                                        <option value="">Seleccionar</option><option value="Presente">Presente</option><option value="Ausente">Ausente</option><option value="Debil">Débil</option>
                                    </select>
                                </div>
                                <div class="input-group">
                                    <label class="label">Sonidos Peristálticos</label>
                                    <select id="sonidos-peristalticos" class="input-field">
                                        <option value="">Seleccionar</option><option value="Presentes">Presentes</option><option value="Ausentes">Ausentes</option>
                                    </select>
                                </div>
                                <div class="input-group">
                                    <label class="label">Extremidades (Pulso Distal)</label>
                                    <select id="pulso-distal-extremidades" class="input-field">
                                        <option value="">Seleccionar</option><option value="Presentes">Presentes</option><option value="Ausentes">Ausentes</option><option value="Debil">Débil</option>
                                    </select>
                                </div>
                                <div class="input-group">
                                    <label class="label">Llenado Capilar</label>
                                    <select id="llenado-capilar" class="input-field">
                                        <option value="">Seleccionar</option><option value="menor2segundos">Menor de 2 segundos</option><option value="mayor3segundos">Mayor de 3 segundos</option>
                                    </select>
                                </div>
                                <div><label class="label">Déficit Neurológico</label><input type="text" class="input-field" placeholder="Consciente, Responde a voz/dolor"></div>
                                <div><label class="label">Exposición / Entorno</label><textarea class="input-field"></textarea></div>
                            </div>
                        </details>
                        
                        <details class="bg-white p-4 rounded-lg shadow mb-4">
                            <summary class="font-semibold text-xl cursor-pointer text-indigo-700">6.12. Auscultación (Corazón y Pulmones)</summary>
                            <p class="mb-4 text-gray-600">Haga clic en un punto de auscultación para obtener información sobre sonidos y patologías.</p>
                            <div class="anatomical-figure border-2 border-gray-300 w-64 h-80 mx-auto relative bg-contain bg-no-repeat bg-center" style="background-image: url('https://placehold.co/300x600/f0f0f0/333333?text=Torso');">
                                <!-- Puntos de auscultación cardíaca -->
                                <div class="auscultation-point" style="top: 15%; left: 35%;" data-point="Foco Aortico" onclick="auscultatePoint('Foco Aortico')">A</div>
                                <div class="auscultation-point" style="top: 15%; left: 65%;" data-point="Foco Pulmonar" onclick="auscultatePoint('Foco Pulmonar')">P</div>
                                <div class="auscultation-point" style="top: 25%; left: 25%;" data-point="Foco Tricuspide" onclick="auscultatePoint('Foco Tricuspide')">T</div>
                                <div class="auscultation-point" style="top: 35%; left: 45%;" data-point="Foco Mitral" onclick="auscultatePoint('Foco Mitral')">M</div>
                                <!-- Puntos de auscultación pulmonar (simplificados) -->
                                <div class="auscultation-point" style="top: 10%; left: 20%;" data-point="Ápice Pulmonar Derecho" onclick="auscultatePoint('Ápice Pulmonar Derecho')">APD</div>
                                <div class="auscultation-point" style="top: 10%; left: 80%;" data-point="Ápice Pulmonar Izquierdo" onclick="auscultatePoint('Ápice Pulmonar Izquierdo')">API</div>
                                <div class="auscultation-point" style="top: 40%; left: 20%;" data-point="Lóbulo Medio Derecho" onclick="auscultatePoint('Lóbulo Medio Derecho')">LMD</div>
                                <div class="auscultation-point" style="top: 40%; left: 80%;" data-point="Lóbulo Inferior Izquierdo" onclick="auscultatePoint('Lóbulo Inferior Izquierdo')">LII</div>
                                <div class="auscultation-point" style="top: 60%; left: 50%;" data-point="Base Pulmonar Central" onclick="auscultatePoint('Base Pulmonar Central')">BPC</div>
                            </div>
                            <div id="auscultation-info" class="mt-4 p-3 bg-indigo-50 rounded-lg font-medium text-indigo-800" style="display:none;">
                                <p class="mb-2"><strong>Punto:</strong> <span id="auscultation-point-name"></span></p>
                                <div id="auscultation-explanation-output" class="mt-2 text-sm bg-gray-100 p-2 rounded"></div>
                                <div id="auscultation-loading" class="mt-2 flex items-center hidden"><div class="spinner"></div><span class="ml-2 text-gray-600">Generando explicación...</span></div>
                                <p class="text-xs text-gray-500 mt-2">La información generada por IA es para fines informativos y de apoyo, NO sustituye el juicio clínico.</p>
                            </div>
                        </details>
                    `
                },
                 'exploracion-fisica': {
                    title: '7. Exploración Física',
                    content: `
                        <p class="mb-4 text-gray-600">Esta sección documenta los hallazgos objetivos de la exploración física, comenzando con los signos vitales y sus alertas automáticas, seguido de un examen sistemático de cabeza a pies.</p>
                        <div class="bg-white p-4 rounded-lg shadow mb-6">
                            <h3 class="font-semibold text-xl mb-4 text-indigo-700">Signos Vitales y Parámetros Cruciales</h3>
                            <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
                                <div class="input-group">
                                    <label class="label">Presión Arterial Sistólica (mmHg)</label>
                                    <input type="number" id="pa-sistolica" class="input-field" oninput="checkVitals()">
                                    <span id="pa-sistolica-alert"></span>
                                </div>
                                <div class="input-group">
                                    <label class="label">Presión Arterial Diastólica (mmHg)</label>
                                    <input type="number" id="pa-diastolica" class="input-field" oninput="checkVitals()">
                                    <span id="pa-diastolica-alert"></span>
                                </div>
                                <div class="input-group">
                                    <label class="label">Frecuencia Cardíaca (lpm)</label>
                                    <input type="number" id="fc" class="input-field" oninput="checkVitals()">
                                    <span id="fc-alert"></span>
                                </div>
                                <div class="input-group">
                                    <label class="label">Frecuencia Respiratoria (rpm)</label>
                                    <input type="number" id="fr" class="input-field" oninput="checkVitals()">
                                    <span id="fr-alert"></span>
                                </div>
                                <div class="input-group">
                                    <label class="label">Temperatura (°C)</label>
                                    <input type="number" step="0.1" id="temp" class="input-field" oninput="checkVitals()">
                                    <span id="temp-alert"></span>
                                </div>
                                <div class="input-group">
                                    <label class="label">Saturación de Oxígeno (%)</label>
                                    <input type="number" id="spo2" class="input-field" oninput="checkVitals()">
                                    <span id="spo2-alert"></span>
                                </div>
                                <div id="spo2-care-alert" class="col-span-full mt-2 hidden p-2 bg-yellow-100 border border-yellow-300 rounded text-sm text-yellow-800"></div>
                                <p class="col-span-full text-xs text-gray-500 mt-2">ALERTA: Lecturas de SpO2 INEXACTAS o ENGAÑOSAS pueden ocurrir en pacientes: HIPOTÉRMICOS, HIPOPERFUSIÓN (SHOCK), INTOXICACIÓN POR CO, ANORMALIDAD DE LA HEMOGLOBINA, ANEMIA, y VASOCONSTRICCIÓN.</p>
                                <div class="input-group">
                                    <label class="label">Glucemia Capilar (mg/dL)</label>
                                    <input type="number" id="glucemia" class="input-field" oninput="checkVitals()">
                                    <span id="glucemia-alert"></span>
                                </div>
                            </div>
                        </div>
                        <div class="bg-white p-4 rounded-lg shadow">
                            <h3 class="font-semibold text-xl mb-4 text-indigo-700">Examinación Secundaria (Cabeza a Pies)</h3>
                            <div class="space-y-4">
                                <details class="bg-gray-50 p-3 rounded-lg">
                                    <summary class="font-semibold cursor-pointer">Cabeza (OPAP)</summary>
                                    <div class="mt-2 text-gray-700">
                                        <p class="font-bold">Observar:</p>
                                        <div class="space-y-1 ml-4">
                                            <div class="flex items-center"><input type="checkbox" id="cabeza-laceraciones" class="mr-2"><label for="cabeza-laceraciones">Laceraciones</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="cabeza-contusiones" class="mr-2"><label for="cabeza-contusiones">Contusiones</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="cabeza-abrasiones" class="mr-2"><label for="cabeza-abrasiones">Abrasiones</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="cabeza-hinchazon" class="mr-2"><label for="cabeza-hinchazon">Hinchazón</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="cabeza-ojosmapache" class="mr-2"><label for="cabeza-ojosmapache">Ojos de Mapache</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="cabeza-signobatalla" class="mr-2"><label for="cabeza-signobatalla">Signo de Batalla</label></div>
                                        </div>
                                        <p class="font-bold mt-3">Palpar:</p>
                                        <div class="space-y-1 ml-4">
                                            <div class="flex items-center"><input type="checkbox" id="cabeza-deformidades" class="mr-2"><label for="cabeza-deformidades">Deformidades</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="cabeza-crepitacion" class="mr-2"><label for="cabeza-crepitacion">Crepitación</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="cabeza-penetracion" class="mr-2"><label for="cabeza-penetracion">Penetración</label></div>
                                        </div>
                                        <p class="font-bold mt-3">Inspeccionar (Ojos y Boca):</p>
                                        <div class="space-y-1 ml-4">
                                            <div class="flex items-center"><input type="checkbox" id="cabeza-drenaje-oidos-nariz" class="mr-2"><label for="cabeza-drenaje-oidos-nariz">Drenaje o sangrado (oídos/nariz)</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="cabeza-boca-anormalidades" class="mr-2"><label for="cabeza-boca-anormalidades">Anormalidades en boca (dientes rotos, objetos extraños, olor a aliento)</label></div>
                                        </div>
                                    </div>
                                </details>
                                <details class="bg-gray-50 p-3 rounded-lg">
                                    <summary class="font-semibold cursor-pointer">Cuello (OPAP)</summary>
                                    <div class="mt-2 text-gray-700">
                                        <p class="font-bold">Observar:</p>
                                        <div class="space-y-1 ml-4">
                                            <div class="flex items-center"><input type="checkbox" id="cuello-desviacion-traqueal" class="mr-2"><label for="cuello-desviacion-traqueal">Desviación traqueal</label></div>
                                            <div class="input-group mt-2">
                                                <label class="label">Venas Yugulares Distendidas (JVD)</label><select id="jvd-present" class="input-field" onchange="toggleJVDDifferentiation()"><option value="">Seleccionar</option><option value="si">Sí</option><option value="no">No</option></select>
                                            </div>
                                            <button id="differentiate-jvd-btn" onclick="differentiateJVD()" class="px-3 py-1 bg-blue-600 text-white rounded-lg hover:bg-blue-700 mt-2 hidden">Diferenciar JVD ✨</button>
                                            <div id="jvd-differentiation-output" class="mt-2 text-sm bg-gray-100 p-2 rounded hidden"></div>
                                            <div id="jvd-loading" class="mt-2 flex items-center hidden"><div class="spinner"></div><span class="ml-2 text-gray-600">Generando explicación...</span></div>
                                            <p class="text-xs text-gray-500 mt-2">La información generada por IA es para fines informativos y de apoyo, NO sustituye el juicio clínico.</p>
                                        </div>
                                        <p class="font-bold mt-3">Palpar:</p>
                                        <div class="space-y-1 ml-4">
                                            <div class="flex items-center"><input type="checkbox" id="cuello-sensibilidad" class="mr-2"><label for="cuello-sensibilidad">Sensibilidad</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="cuello-deformidades" class="mr-2"><label for="cuello-deformidades">Deformidades</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="cuello-crepitacion" class="mr-2"><label for="cuello-crepitacion">Crepitación</label></div>
                                        </div>
                                    </div>
                                </details>
                                <details class="bg-gray-50 p-3 rounded-lg">
                                    <summary class="font-semibold cursor-pointer">Pecho (OPAP)</summary>
                                    <div class="mt-2 text-gray-700">
                                        <p class="font-bold">Observar:</p>
                                        <div class="space-y-1 ml-4">
                                            <div class="flex items-center"><input type="checkbox" id="pecho-heridas" class="mr-2"><label for="pecho-heridas">Heridas</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="pecho-musculos-accesorios" class="mr-2"><label for="pecho-musculos-accesorios">Uso de músculos accesorios</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="pecho-retracciones" class="mr-2"><label for="pecho-retracciones">Retracciones</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="pecho-respiracion-paradojica" class="mr-2"><label for="pecho-respiracion-paradojica">Respiración Paradójica (asociado a fractura de 3+ costillas)</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="pecho-edema-subcutaneo" class="mr-2"><label for="pecho-edema-subcutaneo">Edema Subcutáneo</label></div>
                                            <div class="input-group mt-2"><label class="label">Número de Costillas Fracturadas (si aplica)</label><input type="number" class="input-field"></div>
                                        </div>
                                        <p class="font-bold mt-3">Palpar:</p>
                                        <div class="space-y-1 ml-4">
                                            <div class="flex items-center"><input type="checkbox" id="pecho-crepitacion" class="mr-2"><label for="pecho-crepitacion">Crepitación</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="pecho-sensibilidad-costillas" class="mr-2"><label for="pecho-sensibilidad-costillas">Sensibilidad en costillas</label></div>
                                        </div>
                                        <p class="font-bold mt-3">Auscultar (Sonidos Ventilatorios):</p>
                                        <div class="space-y-1 ml-4">
                                            <div>
                                                <label class="label text-sm font-normal">Disminuidos</label>
                                                <div class="flex flex-wrap gap-x-4">
                                                    <div class="flex items-center"><input type="checkbox" id="sv-disminuidos-der" class="mr-2"><label for="sv-disminuidos-der">Pulmón derecho</label></div>
                                                    <div class="flex items-center"><input type="checkbox" id="sv-disminuidos-izq" class="mr-2"><label for="sv-disminuidos-izq">Pulmón izquierdo</label></div>
                                                    <div class="flex items-center"><input type="checkbox" id="sv-disminuidos-bilateral" class="mr-2"><label for="sv-disminuidos-bilateral">Bilateral</label></div>
                                                </div>
                                            </div>
                                            <div>
                                                <label class="label text-sm font-normal">Aumentados</label>
                                                <div class="flex flex-wrap gap-x-4">
                                                    <div class="flex items-center"><input type="checkbox" id="sv-aumentados-der" class="mr-2"><label for="sv-aumentados-der">Pulmón derecho</label></div>
                                                    <div class="flex items-center"><input type="checkbox" id="sv-aumentados-izq" class="mr-2"><label for="sv-aumentados-izq">Pulmón izquierdo</label></div>
                                                    <div class="flex items-center"><input type="checkbox" id="sv-aumentados-bilateral" class="mr-2"><label for="sv-aumentados-bilateral">Bilateral</label></div>
                                                </div>
                                            </div>
                                            <div>
                                                <label class="label text-sm font-normal">Ausentes</label>
                                                <div class="flex items-center"><input type="checkbox" id="sv-ausentes-der" class="mr-2"><label for="sv-ausentes-der">Pulmón derecho</label></div>
                                                <div class="flex items-center"><input type="checkbox" id="sv-ausentes-izq" class="mr-2"><label for="sv-ausentes-izq">Pulmón izquierdo</label></div>
                                                <div class="flex items-center"><input type="checkbox" id="sv-ausentes-bilateral" class="mr-2"><label for="sv-ausentes-bilateral">Bilateral</label></div>
                                                </div>
                                            </div>
                                        </div>
                                        <p class="font-bold mt-3">Percutir:</p>
                                        <div class="space-y-1 ml-4">
                                            <div class="flex items-center"><input type="checkbox" id="pecho-sonoridad-anormal" class="mr-2"><label for="pecho-sonoridad-anormal">Sonoridad pulmonar anormal</label></div>
                                        </div>
                                    </div>
                                </details>
                                <details class="bg-gray-50 p-3 rounded-lg">
                                    <summary class="font-semibold cursor-pointer">Abdomen (OPAP)</summary>
                                    <div class="mt-2 text-gray-700">
                                        <p class="font-bold">Observar:</p>
                                        <div class="space-y-1 ml-4">
                                            <div class="flex items-center"><input type="checkbox" id="abdomen-distension" class="mr-2"><label for="abdomen-distension">Distensión</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="abdomen-heridas" class="mr-2"><label for="abdomen-heridas">Heridas</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="abdomen-cicatrices" class="mr-2"><label for="abdomen-cicatrices">Cicatrices</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="abdomen-equimosis-check" class="mr-2"><label for="abdomen-equimosis-check">Equimosis</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="abdomen-tabla-check" class="mr-2"><label for="abdomen-tabla-check">Rigidez en Tabla</label></div>
                                        </div>
                                        <p class="font-bold mt-3">Auscultar:</p>
                                        <div class="space-y-1 ml-4">
                                            <div class="flex items-center"><input type="checkbox" id="abdomen-ruidos-hidroaereos-anormales" class="mr-2"><label for="abdomen-ruidos-hidroaereos-anormales">Ruidos hidroaéreos anormales</label></div>
                                        </div>
                                        <p class="font-bold mt-3">Percutir:</p>
                                        <div class="space-y-1 ml-4">
                                            <div class="flex items-center"><input type="checkbox" id="abdomen-matidez" class="mr-2"><label for="abdomen-matidez">Matidez</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="abdomen-timpanismo" class="mr-2"><label for="abdomen-timpanismo">Timpanismo</label></div>
                                        </div>
                                        <p class="font-bold mt-3">Palpar:</p>
                                        <div class="space-y-1 ml-4">
                                            <div class="flex items-center"><input type="checkbox" id="abdomen-dolor-palpacion" class="mr-2"><label for="abdomen-dolor-palpacion">Dolor a la palpación</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="abdomen-masas" class="mr-2"><label for="abdomen-masas">Masas</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="abdomen-rigidez" class="mr-2"><label for="abdomen-rigidez">Rigidez</label></div>
                                        </div>
                                    </div>
                                </details>
                                <details class="bg-gray-50 p-3 rounded-lg">
                                    <summary class="font-semibold cursor-pointer">Pelvis y Genitourinario (OPAP)</summary>
                                    <div class="mt-2 text-gray-700">
                                        <p class="font-bold">Observar:</p>
                                        <div class="space-y-1 ml-4">
                                            <div class="flex items-center"><input type="checkbox" id="pelvis-incontinencia" class="mr-2"><label for="pelvis-incontinencia">Incontinencia</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="pelvis-priapismo" class="mr-2"><label for="pelvis-priapismo">Priapismo</label></div>
                                        </div>
                                        <p class="font-bold mt-3">Palpar:</p>
                                        <div class="space-y-1 ml-4">
                                            <div class="flex items-center"><input type="checkbox" id="pelvis-dolor-palpacion" class="mr-2"><label for="pelvis-dolor-palpacion">Dolor a la palpación</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="pelvis-deformidad" class="mr-2"><label for="pelvis-deformidad">Deformidad</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="pelvis-crepitacion" class="mr-2"><label for="pelvis-crepitacion">Crepitación</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="pelvis-contusion" class="mr-2"><label for="pelvis-contusion">Contusión</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="pelvis-abrasion" class="mr-2"><label for="pelvis-abrasion">Abrasión</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="pelvis-penetration" class="mr-2"><label for="pelvis-penetration">Penetración</label></div>
                                        </div>
                                    </div>
                                </details>
                                <details class="bg-gray-50 p-3 rounded-lg">
                                    <summary class="font-semibold cursor-pointer">Extremidades (OPAP)</summary>
                                    <div class="mt-2 text-gray-700">
                                        <p class="font-bold">Observar:</p>
                                        <div class="space-y-1 ml-4">
                                            <div class="flex items-center"><input type="checkbox" id="ext-deformidad" class="mr-2"><label for="ext-deformidad">Deformidad</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="ext-heridas" class="mr-2"><label for="ext-heridas">Heridas</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="ext-inflamacion" class="mr-2"><label for="ext-inflamacion">Inflamación</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="ext-cambios-color" class="mr-2"><label for="ext-cambios-color">Cambios de color</label></div>
                                        </div>
                                        <p class="font-bold mt-3">Palpar:</p>
                                        <div class="space-y-1 ml-4">
                                            <div class="flex items-center"><input type="checkbox" id="ext-dolor" class="mr-2"><label for="ext-dolor">Dolor</label></div>
                                            <div class="input-group">
                                                <label class="label text-sm font-normal">Pulsos Distales</label>
                                                <select id="pulso-distal-extremidades-ef" class="input-field">
                                                    <option value="">Seleccionar</option><option value="Presentes">Presentes</option><option value="Ausentes">Ausentes</option><option value="Debil">Débil</option>
                                                </select>
                                            </div>
                                            <div class="flex items-center"><input type="checkbox" id="ext-temperatura-anormal" class="mr-2"><label for="ext-temperatura-anormal">Temperatura anormal</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="ext-crepitacion" class="mr-2"><label for="ext-crepitacion">Crepitación</label></div>
                                        </div>
                                        <p class="font-bold mt-3">Evaluar:</p>
                                        <div class="space-y-1 ml-4">
                                            <div class="flex items-center"><input type="checkbox" id="ext-funcion-motora-alterada" class="mr-2"><label for="ext-funcion-motora-alterada">Función motora alterada</label></div>
                                            <div class="flex items-center"><input type="checkbox" id="ext-sensibilidad-alterada" class="mr-2"><label for="ext-sensibilidad-alterada">Sensibilidad alterada (parestesias)</label></div>
                                            <div class="input-group mt-2">
                                                <label class="label text-sm font-normal">Llenado Capilar</label>
                                                <select id="llenado-capilar-ef" class="input-field">
                                                    <option value="">Seleccionar</option><option value="menor2segundos">Menor de 2 segundos</option><option value="mayor3segundos">Mayor de 3 segundos</option>
                                                </select>
                                            </div>
                                        </div>
                                    </div>
                                </details>
                            </div>
                        </div>
                    `
                },
                 'evaluaciones-especificas': {
                    title: '8. Evaluaciones Específicas y Escalas de Emergencia',
                    content: `
                        <p class="mb-4 text-gray-600">Esta sección contiene herramientas especializadas para la evaluación en contextos de emergencia como pediatría, trauma y quemaduras. Las calculadoras y alertas están diseñadas para agilizar la toma de decisiones críticas.</p>
                        <!-- Guía de Evaluación Inicial -->
                        <div class="bg-white p-4 rounded-lg shadow mb-6">
                            <h3 class="font-semibold text-xl mb-4 text-indigo-700">8.4. Guía de Evaluación Inicial (Paciente con Trauma vs. Médico)</h3>
                            <p class="mb-4 text-gray-600">Esta guía ayuda a diferenciar la aproximación inicial al paciente basándose en el mecanismo de lesión (MOI) o la respuesta general.</p>
                            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                                <div>
                                    <h4 class="font-bold text-lg text-red-700 mb-2">PACIENTE CON TRAUMA</h4>
                                    <p class="font-semibold text-sm mb-1">Mecanismo de Lesión (MOI) Significativo:</p>
                                    <ul class="list-disc list-inside text-sm mb-3 pl-4">
                                        <li>Evaluación Rápida de Trauma</li>
                                        <li>D: Cabeza, Crepitación</li>
                                        <li>C: Tórax, Crepitación, Respiración, Movimiento Paradójico, Sonidos Respiratorios</li>
                                        <li>A: Abdomen, Rigidez, Distensión</li>
                                        <li>P: Pelvis/GU, Dolor al Movimiento</li>
                                        <li>B: Extremidades, Pulso/Motor/Sensorial</li>
                                        <li>T: Columna Torácica (parte posterior), verificar T</li>
                                        <li>L: Columna Lumbar (parte posterior), verificar L</li>
                                        <li>S: Columna Sacra (parte posterior), verificar S</li>
                                    </ul>
                                    <p class="font-semibold text-sm mb-1">MOI No Significativo:</p>
                                    <ul class="list-disc list-inside text-sm pl-4">
                                        <li>Determinar Queja Principal</li>
                                        <li>Realizar Examen Focalizado de la Lesión y Áreas Compatibles con el MOI dado.</li>
                                    </ul>
                                    <p class="font-semibold text-sm mt-3">Signos Vitales Basales</p>
                                    <p class="font-semibold text-sm mt-3">Obtener Historial SAMPLE:</p>
                                    <ul class="list-disc list-inside text-sm pl-4">
                                        <li>S: Signos y Síntomas</li>
                                        <li>A: Alergias</li>
                                        <li>M: Medicamentos</li>
                                        <li>P: Historial Pertinente</li>
                                        <li>L: Última Ingesta Oral</li>
                                        <li>E: Eventos Previos</li>
                                    </ul>
                                </div>
                                <div>
                                    <h4 class="font-bold text-lg text-blue-700 mb-2">PACIENTE MÉDICO</h4>
                                    <p class="font-semibold text-sm mb-1">Paciente Inconsciente:</p>
                                    <ul class="list-disc list-inside text-sm mb-3 pl-4">
                                        <li>Examen Físico Rápido</li>
                                        <li>D: Cabeza, Cuello, Distensión Yugular (JVD), Dispositivo de Alerta Médica</li>
                                        <li>C: Tórax, Sonidos Respiratorios</li>
                                        <li>A: Abdomen, Rigidez, Distensión</li>
                                        <li>P: Pelvis/GU</li>
                                        <li>B: Sangre, Orina, Heces</li>
                                        <li>M: Dispositivo de Alerta Médica</li>
                                        <li>S: Posterior</li>
                                    </ul>
                                    <p class="font-semibold text-sm mb-1">Paciente Consciente:</p>
                                    <ul class="list-disc list-inside text-sm mb-3 pl-4">
                                        <li>Obtener Historial del Episodio:</li>
                                        <ul class="list-circle list-inside ml-4">
                                            <li>Inicio (Onset)</li>
                                            <li>Provocación</li>
                                            <li>Calidad</li>
                                            <li>Irradiación</li>
                                            <li>Severidad</li>
                                            <li>Tiempo</li>
                                        </ul>
                                    </ul>
                                    <p class="font-semibold text-sm mt-3">Signos Vitales Basales</p>
                                    <p class="font-semibold text-sm mt-3">Obtener Historial SAMPLE</p>
                                    <p class="font-semibold text-sm mt-3">Examen Físico Focalizado DCAPBTLS</p>
                                    <p class="text-sm mt-1">Revisar áreas sugeridas por MOI y SAMPLE.</p>
                                </div>
                            </div>
                            <p class="mt-6 font-bold text-sm text-gray-700">CONSIDERAR SVA, REALIZAR INTERVENCIONES Y TRANSPORTAR.</p>
                        </div>

                        <!-- Quemaduras -->
                        <div class="bg-white p-4 rounded-lg shadow mb-6">
                             <h3 class="font-semibold text-xl mb-4 text-indigo-700">Evaluación de Quemaduras (Regla del 9 y Parkland)</h3>
                             <div class="flex flex-col md:flex-row gap-8">
                                <div class="flex-1 space-y-4">
                                   <div>
                                        <h4 class="font-semibold text-center">Adulto</h4>
                                        <div class="p-2 border rounded-lg bg-gray-50 text-center text-sm">
                                            Cabeza: 9%<br>
                                            Cada Brazo: 9%<br>
                                            Tronco Ant: 18%<br>
                                            Tronco Post: 18%<br>
                                            Cada Pierna: 18%<br>
                                            Genitales: 1%
                                        </div>
                                   </div>
                                   <div>
                                        <h4 class="font-semibold text-center">Pediátrico</h4>
                                        <div class="p-2 border rounded-lg bg-gray-50 text-center text-sm">
                                            Cabeza: 18%<br>
                                            Cada Brazo: 9%<br>
                                            Tronco Ant: 18%<br>
                                            Tronco Post: 18%<br>
                                            Cada Pierna: 13.5%<br>
                                            Genitales: 1%
                                        </div>
                                   </div>
                                </div>
                                <div class="flex-1">
                                    <div class="input-group">
                                        <label class="label">Peso del Paciente (kg)</label>
                                        <input type="number" id="burn-weight" class="input-field" oninput="calculateParkland()">
                                    </div>
                                    <div class="input-group">
                                        <label class="label">Superficie Corporal Quemada (%)</label>
                                        <input type="number" id="burn-bsa" class="input-field" oninput="calculateParkland()">
                                    </div>
                                    <div id="parkland-results" class="mt-4 p-3 bg-indigo-50 rounded-lg space-y-2" style="display:none;">
                                        <h4 class="font-semibold text-indigo-800">Fórmula de Parkland (Líquidos 24h)</h4>
                                        <p>Volumen Total: <span id="parkland-total" class="font-bold">0</span> ml</p>
                                        <p>Primeras 8h: <span id="parkland-first-8h" class="font-bold">0</span> ml</p>
                                        <p>Siguientes 16h: <span id="parkland-next-16h" class="font-bold">0</span> ml</p>
                                        <p id="burn-alert" class="mt-2 font-bold"></p>
                                    </div>
                                </div>
                             </div>
                        </div>
                        <!-- Maniquí Interactivo -->
                        <div class="bg-white p-4 rounded-lg shadow">
                            <h3 class="font-semibold text-xl mb-4 text-indigo-700">Maniquí Interactivo (Marcar Lesiones)</h3>
                            <p class="mb-4 text-gray-600">Haga clic en la ubicación de la lesión en el maniquí. Describa el hallazgo.</p>
                            <div class="flex flex-col md:flex-row gap-4 justify-around items-center">
                                <div class="anatomical-figure border rounded-lg overflow-hidden relative" style="width:150px; height:300px; background-image:url('https://placehold.co/150x300/f0f0f0/333333?text=Maniqui+Frontal'); background-size:contain; background-repeat:no-repeat; background-position:center;" data-view="frontal" onclick="markLesion(event, this.dataset.view)">
                                    <h4 class="absolute top-1 left-1/2 -translate-x-1/2 text-xs font-bold text-gray-700">Frontal</h4>
                                    </div>
                                <div class="anatomical-figure border rounded-lg overflow-hidden relative" style="width:150px; height:300px; background-image:url('https://placehold.co/150x300/f0f0f0/333333?text=Maniqui+Posterior'); background-size:contain; background-repeat:no-repeat; background-position:center;" data-view="posterior" onclick="markLesion(event, this.dataset.view)">
                                    <h4 class="absolute top-1 left-1/2 -translate-x-1/2 text-xs font-bold text-gray-700">Posterior</h4>
                                </div>
                            </div>
                            <div id="lesion-list" class="mt-4 p-3 bg-gray-100 rounded-lg">
                                <h4 class="font-semibold mb-2">Lesiones Marcadas:</h4>
                                </div>
                        </div>
                    `
                },
                'plan-manejo': {
                    title: '9. Plan de Manejo y Prescripción',
                    content: `
                        <p class="mb-4 text-gray-600">Establezca el plan de tratamiento integral del paciente, incluyendo diagnósticos, farmacoterapia, medidas no farmacológicas y plan de seguimiento. Integración con calculadoras para precisión.</p>
                        <h3 class="font-semibold text-xl mb-4 text-indigo-700">Impresión Diagnóstica</h3>
                        <div class="space-y-2">
                            <div><label class="label">Diagnóstico Principal</label><input type="text" id="diag-principal" class="input-field"></div>
                            <div><label class="label">Diagnósticos Secundarios / Comorbilidades</label><textarea id="diag-secundarios" class="input-field"></textarea></div>
                            <div><label class="label">Diagnósticos Diferenciales</label><textarea id="diag-diferenciales" class="input-field"></textarea></div>
                        </div>
                         <!-- Asistente de Diagnóstico Diferencial -->
                        <div class="bg-gray-100 p-4 rounded-lg shadow mt-6">
                            <h3 class="font-semibold text-xl text-indigo-700">Asistente de Diagnóstico Diferencial ✨</h3>
                            <p class="mb-4 text-gray-600">Ingrese los síntomas y hallazgos clave del paciente para obtener posibles diagnósticos diferenciales sugeridos por IA.</p>
                            <div class="input-group">
                                <label class="label">Síntomas y Hallazgos Clave</label>
                                <textarea id="symptoms-for-diagnosis" class="input-field" rows="4" placeholder="Ej: Dolor torácico opresivo, disnea, sudoración fría, antecedente de HTA"></textarea>
                            </div>
                            <button onclick="generateDifferentialDiagnosis()" id="generate-diagnosis-btn" class="px-4 py-2 bg-green-600 text-white rounded-lg hover:bg-green-700">✨ Generar Diagnósticos Diferenciales</button>
                            <div id="diagnosis-loading" class="mt-4 flex items-center hidden">
                                <div class="spinner"></div>
                                <span class="ml-2 text-gray-600">Generando diagnósticos...</span>
                            </div>
                            <div id="ai-diagnosis-output" class="mt-4 p-4 bg-gray-50 rounded-lg border border-gray-200 text-gray-700 whitespace-pre-wrap">
                            </div>
                            <p class="text-xs text-gray-500 mt-2">La información generada por IA es para fines informativos y de apoyo, NO sustituye el juicio clínico.</p>
                        </div>
                        <!-- Refinador de Diagnóstico Diferencial -->
                        <div class="bg-gray-100 p-4 rounded-lg shadow mt-6">
                            <h3 class="font-semibold text-xl text-indigo-700">Refinador de Diagnóstico Diferencial ✨</h3>
                            <p class="mb-4 text-gray-600">Ingrese un diagnóstico sospechado para obtener características clave de confirmación/descarte o diferenciación.</p>
                            <div class="input-group">
                                <label class="label">Diagnóstico Sospechado</label>
                                <textarea id="diagnosis-refiner-input" class="input-field" rows="3" placeholder="Ej: Infarto agudo de miocardio, Apendicitis aguda"></textarea>
                            </div>
                            <button onclick="refineDiagnosis()" id="refine-diagnosis-btn" class="px-4 py-2 bg-yellow-600 text-white rounded-lg hover:bg-yellow-700">✨ Refinar Diagnóstico</button>
                            <div id="refiner-loading" class="mt-4 flex items-center hidden">
                                <div class="spinner"></div>
                                <span class="ml-2 text-gray-600">Refinando diagnóstico...</span>
                            </div>
                            <div id="ai-refiner-output" class="mt-4 p-4 bg-gray-50 rounded-lg border border-gray-200 text-gray-700 whitespace-pre-wrap">
                                <!-- La refinación de diagnóstico aparecerá aquí -->
                            </div>
                            <p class="text-xs text-gray-500 mt-2">La información generada por IA es para fines informativos y de apoyo, NO sustituye el juicio clínico.</p>
                        </div>


                        <h3 class="font-semibold text-xl mb-4 mt-6 text-indigo-700">Plan de Tratamiento Farmacológico</h3>
                        <div class="space-y-2">
                            <div id="medication-table-body">
                                <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4 medication-row border-b pb-4 mb-4">
                                    <div class="input-group"><label class="label">Medicamento</label><input type="text" class="input-field medication-name"></div>
                                    <div class="input-group"><label class="label">Dosis</label><input type="text" class="input-field medication-dose"></div>
                                    <div class="input-group"><label class="label">Vía</label><input type="text" class="input-field medication-via"></div>
                                    <div class="input-group"><label class="label">Frecuencia</label><input type="text" class="input-field medication-frequency" placeholder="ej. Cada 8h, QD, BID"></div>
                                    <div class="input-group"><label class="label">Última Dosis (HH:MM)</label><input type="time" class="input-field medication-last-time"></div>
                                    <div class="input-group"><label class="label">Próxima Dosis</label><input type="text" class="input-field medication-next-time" readonly></div>
                                    <button onclick="getDrugDetails(this)" class="px-3 py-1 bg-blue-600 text-white rounded-lg hover:bg-blue-700 text-sm md:col-span-2 mt-4 md:mt-0">Detalles del Medicamento ✨</button>
                                    <div class="col-span-full mt-2 hidden drug-details-output bg-gray-100 p-2 rounded text-sm relative">
                                        <div class="spinner absolute top-1 right-1 hidden"></div>
                                        <p class="text-xs text-gray-500 mt-2">La información generada por IA es para fines informativos y de apoyo, NO sustituye el juicio clínico.</p>
                                    </div>
                                </div>
                            </div>
                            <button onclick="addMedicationRow()" class="px-4 py-2 bg-indigo-600 text-white rounded-lg hover:bg-indigo-700">Añadir Medicamento</button>
                        </div>
                        <h4 class="font-semibold text-lg mb-2 mt-6 text-indigo-700">Calculadoras de Dosificación</h4>
                        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                            <div class="bg-gray-100 p-4 rounded-lg">
                                <h5 class="font-semibold mb-2">Dosis Pediátrica/Por Peso</h5>
                                <div><label class="label">Dosis Deseada (mg/kg)</label><input type="number" id="calc-dosis-deseada" class="input-field"></div>
                                <div><label class="label">Peso (kg)</label><input type="number" id="calc-dosis-peso" class="input-field"></div>
                                <div><label class="label">Concentración (mg/ml)</label><input type="number" id="calc-dosis-concentracion" class="input-field"></div>
                                <div class="mt-2"><label class="label">Dosis a Administrar (ml): <span id="result-dosis" class="font-bold">0</span></label></div>
                                <button onclick="calculateDose()" class="px-3 py-1 bg-indigo-500 text-white rounded-lg hover:bg-indigo-600 mt-2">Calcular</button>
                            </div>
                            <div class="bg-gray-100 p-4 rounded-lg">
                                <h5 class="font-semibold mb-2">Goteo de Soluciones IV (gotas/min)</h5>
                                <div><label class="label">Volumen Total (ml)</label><input type="number" id="calc-goteo-volumen" class="input-field"></div>
                                <div><label class="label">Tiempo (horas)</label><input type="number" id="calc-goteo-tiempo" class="input-field"></div>
                                <div><label class="label">Factor Goteo (gotas/ml)</label><select id="calc-goteo-factor" class="input-field"><option value="10">10 (Macrogotero)</option><option value="15">15</option><option value="20">20</option><option value="60">60 (Microgotero)</option></select></div>
                                <div class="mt-2"><label class="label">Velocidad de Goteo (gotas/min): <span id="result-goteo" class="font-bold">0</span></label></div>
                                <button onclick="calculateDripRate()" class="px-3 py-1 bg-indigo-500 text-white rounded-lg hover:bg-indigo-600 mt-2">Calcular</button>
                            </div>
                        </div>
                        <h3 class="font-semibold text-xl mb-4 mt-6 text-indigo-700">Restablecimiento de Fluidos en Deshidratación</h3>
                        <div class="space-y-2">
                             <div><label class="label">Grado de Deshidratación</label><select id="dehydration-degree" class="input-field"><option value="0">Seleccionar</option><option value="5">Leve (3-5%)</option><option value="7.5">Moderada (6-9%)</option><option value="10">Grave (>10%)</option></select></div>
                             <div><label class="label">Peso Actual (kg)</label><input type="number" id="dehydration-weight" class="input-field" oninput="calculateDehydration()"></div>
                             <div class="mt-2"><label class="label">Déficit Estimado (ml): <span id="dehydration-deficit" class="font-bold">0</span></label></div>
                             <div class="mt-2"><label class="label">Necesidades Basales (ml/24h): <span id="dehydration-basal" class="font-bold">0</span></label></div>
                             <div class="mt-2"><label class="label">Volumen Total 24h (ml): <span id="dehydration-total-24h" class="font-bold">0</span></label></div>
                             <button onclick="calculateDehydration()" class="px-3 py-1 bg-indigo-500 text-white rounded-lg hover:bg-indigo-600 mt-2">Calcular Hidratación</button>
                        </div>

                        <!-- Generador de Material Educativo para Pacientes -->
                        <div class="bg-indigo-50 p-4 rounded-lg shadow mt-6">
                            <h3 class="font-semibold text-xl text-indigo-700">Generador de Material Educativo para Pacientes ✨</h3>
                            <p class="mb-4 text-gray-600">Genere explicaciones sencillas sobre condiciones o tratamientos médicos para educar a sus pacientes.</p>
                            <div class="input-group">
                                <label class="label">Condición o Tratamiento (para explicar al paciente)</label>
                                <textarea id="patient-education-input" class="input-field" rows="4" placeholder="Ej: Diabetes tipo 2, uso de inhalador, manejo de la hipertensión"></textarea>
                            </div>
                            <button onclick="generatePatientEducation()" id="generate-patient-education-btn" class="px-4 py-2 bg-pink-600 text-white rounded-lg hover:bg-pink-700">✨ Generar Material Educativo</button>
                            <div id="patient-education-loading" class="mt-4 flex items-center hidden">
                                <div class="spinner"></div>
                                <span class="ml-2 text-gray-600">Generando material...</span>
                            </div>
                            <div id="ai-patient-education-output" class="mt-4 p-4 bg-gray-100 rounded-lg border border-gray-200 text-gray-700 whitespace-pre-wrap">
                                <!-- El material educativo aparecerá aquí -->
                            </div>
                            <p class="text-xs text-gray-500 mt-2">La información generada por IA es para fines informativos y de apoyo, NO sustituye el juicio clínico.</p>
                        </div>
                    `
                },
                'registro': {
                    title: '10. Registro, Anexos y Firma',
                    content: `
                        <p class="mb-4 text-gray-600">Finalice el registro identificándose como el profesional responsable. Puede adjuntar documentos relevantes como resultados de laboratorio o imágenes, y firmar electrónicamente para validar la historia clínica.</p>
                        <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                            <div class="bg-white p-4 rounded-lg shadow space-y-4">
                                <h3 class="font-semibold text-xl text-indigo-700">Identificación del Examinador</h3>
                                <div><label class="label">Nombre Completo</label><input type="text" id="examiner-name" class="input-field"></div>
                                <div><label class="label">Número de Licencia / Cédula</label><input type="text" id="examiner-license" class="input-field"></div>
                                <div><label class="label">Especialidad</label><input type="text" id="examiner-specialty" class="input-field"></div>
                            </div>
                             <div class="bg-white p-4 rounded-lg shadow space-y-2">
                                <h3 class="font-semibold text-xl text-indigo-700">Firma Electrónica</h3>
                                <canvas id="signature-pad" class="w-full h-40 bg-gray-50"></canvas>
                                <button id="clear-signature" class="px-4 py-2 bg-gray-200 hover:bg-gray-300 rounded-lg text-sm">Limpiar</button>
                            </div>
                        </div>
                        <div class="bg-white p-4 rounded-lg shadow mt-6">
                            <h3 class="font-semibold text-xl text-indigo-700">Anexos / Documentos Adjuntos</h3>
                            <div class="mt-4 border-2 border-dashed border-gray-300 rounded-lg p-6 text-center">
                                <p class="mb-2 text-gray-500">Arrastre archivos o</p>
                                <input type="file" id="file-upload" multiple class="hidden">
                                <button onclick="document.getElementById('file-upload').click()" class="px-4 py-2 bg-indigo-600 text-white rounded-lg hover:bg-indigo-700">Seleccionar Archivos</button>
                                <div id="file-list" class="mt-4 text-left"></div>
                            </div>
                        </div>

                        <div class="bg-white p-4 rounded-lg shadow mt-6">
                            <h3 class="font-semibold text-xl text-indigo-700">Resumen de Historia Clínica ✨</h3>
                            <p class="mb-4 text-gray-600">Genere un resumen conciso de los datos clave de la historia clínica utilizando inteligencia artificial.</p>
                            <button onclick="generateSummary()" id="generate-summary-btn" class="px-4 py-2 bg-purple-600 text-white rounded-lg hover:bg-purple-700">✨ Generar Resumen</button>
                            <div id="summary-loading" class="mt-4 flex items-center hidden">
                                <div class="spinner"></div>
                                <span class="ml-2 text-gray-600">Generando resumen...</span>
                            </div>
                            <div id="ai-summary-output" class="mt-4 p-4 bg-gray-100 rounded-lg border border-gray-200 text-gray-700 whitespace-pre-wrap">
                                </div>
                            <p class="text-xs text-gray-500 mt-2">La información generada por IA es para fines informativos y de apoyo, NO sustituye el juicio clínico.</p>
                        </div>

                        <div class="bg-white p-4 rounded-lg shadow mt-6">
                            <h3 class="font-semibold text-xl text-indigo-700">Explicador de Términos Médicos ✨</h3>
                            <p class="mb-4 text-gray-600">Obtenga una explicación sencilla de términos médicos para pacientes, impulsada por IA.</p>
                            <div class="input-group">
                                <label class="label">Término/Concepto a Explicar</label>
                                <textarea id="medical-term-input" class="input-field" rows="3" placeholder="Ej: Hipertensión, Miocardiopatía, Glucemia Capilar"></textarea>
                            </div>
                            <button onclick="explainMedicalTerm()" id="explain-term-btn" class="px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700">✨ Explicar Término</button>
                            <div id="explanation-loading" class="mt-4 flex items-center hidden">
                                <div class="spinner"></div>
                                <span class="ml-2 text-gray-600">Generando explicación...</span>
                            </div>
                            <div id="ai-explanation-output" class="mt-4 p-4 bg-gray-100 rounded-lg border border-gray-200 text-gray-700 whitespace-pre-wrap">
                                </div>
                            <p class="text-xs text-gray-500 mt-2">La información generada por IA es para fines informativos y de apoyo, NO sustituye el juicio clínico.</p>
                        </div>
                    `
                }
            };
            
            const mainContent = document.getElementById('main-content');
            const navMenu = document.getElementById('navigation-menu');

            function renderSection(id) {
                const section = sections[id];
                if (!section) return;

                mainContent.innerHTML = `
                    <div class="bg-white p-6 rounded-lg shadow-lg">
                        <h2 class="text-2xl font-bold mb-4">${section.title}</h2>
                        <div class="content-body">${section.content}</div>
                    </div>
                `;
                
                document.querySelectorAll('.nav-link').forEach(link => {
                    link.classList.toggle('active', link.getAttribute('href') === `#${id}`);
                });

                if (id === 'registro') {
                    const canvas = document.getElementById('signature-pad');
                    const signaturePad = new SignaturePad(canvas);
                    document.getElementById('clear-signature').addEventListener('click', () => {
                        signaturePad.clear();
                    });
                     const fileUpload = document.getElementById('file-upload');
                    const fileList = document.getElementById('file-list');
                    fileUpload.addEventListener('change', (event) => {
                        fileList.innerHTML = ''; // Clear list
                        for (const file of event.target.files) {
                            const fileItem = document.createElement('div');
                            fileItem.textContent = file.name;
                            fileList.appendChild(fileItem);
                        }
                    });
                }
                
                // Re-attach specific section listeners
                if (id === 'inicio') {
                    setupEmojiSelector('mood-emoji-selector', 'mood-scale');
                } else if (id === 'hea') {
                    setupEmojiSelector('intensity-emoji-selector', 'hea-intensity');
                } else if (id === 'antecedentes') {
                    document.getElementById('habito-alcohol').addEventListener('change', () => toggleHabitDetail('alcohol'));
                    document.getElementById('habito-tabaco').addEventListener('change', () => toggleHabitDetail('tabaco'));
                    document.getElementById('habito-drogas').addEventListener('change', () => toggleHabitDetail('drogas'));
                    toggleHabitDetail('alcohol'); // Initial call to set visibility
                    toggleHabitDetail('tabaco');
                    toggleHabitDetail('drogas');
                } else if (id === 'revision-sistemas') {
                    document.querySelectorAll('.quadrant').forEach(quadrant => {
                        quadrant.addEventListener('click', (e) => showOrgansNineQuadrants(e.target.id));
                    });
                    document.querySelectorAll('.auscultation-point').forEach(point => {
                        point.addEventListener('click', (e) => auscultatePoint(e.target.dataset.point));
                    });
                    document.getElementById('gcs-ocular').addEventListener('change', calculateGCS);
                    document.getElementById('gcs-verbal').addEventListener('change', calculateGCS);
                    document.getElementById('gcs-motor').addEventListener('change', calculateGCS);
                    calculateGCS(); // Initial calculation for GCS
                    document.getElementById('osteo-mialgias').addEventListener('change', () => toggleOsteoDetail('mialgias'));
                    document.getElementById('osteo-artralgia').addEventListener('change', () => toggleOsteoDetail('artralgia'));
                    toggleOsteoDetail('mialgias'); // Initial calls
                    toggleOsteoDetail('artralgia');
                    document.getElementById('hematoma-present').addEventListener('change', toggleHematomaDetails);
                    toggleHematomaDetails();
                    document.getElementById('hemorragia-present').addEventListener('change', toggleHemorrhageDetails);
                    toggleHemorrhageDetails();
                    document.getElementById('endo-hipertiroidismo').addEventListener('change', () => toggleEndocrineDetail('hipertiroidismo'));
                    document.getElementById('endo-hipotiroidismo').addEventListener('change', () => toggleEndocrineDetail('hipotiroidismo'));
                    toggleEndocrineDetail('hipertiroidismo'); // Initial calls
                    toggleEndocrineDetail('hipotiroidismo');
                } else if (id === 'exploracion-fisica') {
                    document.querySelectorAll('#exploracion-fisica input[type="number"]').forEach(input => {
                        input.addEventListener('input', checkVitals);
                    });
                    const jvdSelect = document.getElementById('jvd-present');
                    if (jvdSelect) jvdSelect.addEventListener('change', toggleJVDDifferentiation);
                    const differentiateBtn = document.getElementById('differentiate-jvd-btn');
                    if (differentiateBtn) differentiateBtn.onclick = differentiateJVD;
                    toggleJVDDifferentiation(); // Initial call
                } else if (id === 'evaluaciones-especificas') {
                    const burnWeight = document.getElementById('burn-weight');
                    const burnBsa = document.getElementById('burn-bsa');
                    if (burnWeight) burnWeight.addEventListener('input', calculateParkland);
                    if (burnBsa) burnBsa.addEventListener('input', calculateParkland);
                    if (burnWeight) calculateParkland(); // Initial calculation for Parkland
                    document.querySelectorAll('.anatomical-figure').forEach(figure => {
                        figure.onclick = (event) => markLesion(event, figure.dataset.view);
                    });
                } else if (id === 'plan-manejo') {
                    const calcDosisDeseada = document.getElementById('calc-dosis-deseada');
                    const calcDosisPeso = document.getElementById('calc-dosis-peso');
                    const calcDosisConcentracion = document.getElementById('calc-dosis-concentracion');
                    const calcGoteoVolumen = document.getElementById('calc-goteo-volumen');
                    const calcGoteoTiempo = document.getElementById('calc-goteo-tiempo');
                    const calcGoteoFactor = document.getElementById('calc-goteo-factor');
                    const dehydrationDegree = document.getElementById('dehydration-degree');
                    const dehydrationWeight = document.getElementById('dehydration-weight');

                    if (calcDosisDeseada) calcDosisDeseada.addEventListener('input', calculateDose);
                    if (calcDosisPeso) calcDosisPeso.addEventListener('input', calculateDose);
                    if (calcDosisConcentracion) calcDosisConcentracion.addEventListener('input', calculateDose);
                    if (calcGoteoVolumen) calcGoteoVolumen.addEventListener('input', calculateDripRate);
                    if (calcGoteoTiempo) calcGoteoTiempo.addEventListener('input', calculateDripRate);
                    if (calcGoteoFactor) calcGoteoFactor.addEventListener('change', calculateDripRate);
                    if (dehydrationDegree) dehydrationDegree.addEventListener('change', calculateDehydration);
                    if (dehydrationWeight) dehydrationWeight.addEventListener('input', calculateDehydration);
                    
                    if (calcDosisDeseada) calculateDose();
                    if (calcGoteoVolumen) calculateDripRate();
                    if (dehydrationDegree) calculateDehydration();

                    document.querySelectorAll('.medication-name').forEach(input => input.addEventListener('input', calculateNextMedTime));
                    document.querySelectorAll('.medication-frequency').forEach(input => input.addEventListener('input', calculateNextMedTime));
                    document.querySelectorAll('.medication-last-time').forEach(input => input.addEventListener('input', calculateNextMedTime));
                }
            }
            
            navMenu.addEventListener('click', e => {
                if (e.target.tagName === 'A') {
                    e.preventDefault();
                    const id = e.target.getAttribute('href').substring(1);
                    renderSection(id);
                    window.history.pushState({ id }, '', `#${id}`);
                }
            });

            window.onpopstate = (e) => {
                if(e.state) {
                    renderSection(e.state.id);
                }
            };

            const initialId = window.location.hash ? window.location.hash.substring(1) : 'datos-generales';
            renderSection(initialId);
        });
        
        // Functions for interactivity
        function setupEmojiSelector(selectorId, targetInputId) {
            const selector = document.getElementById(selectorId);
            const targetInput = document.getElementById(targetInputId);

            if (!selector || !targetInput) {
                //console.warn(`Emoji selector or target input not found for ${selectorId}. Skipping setup.`);
                return;
            }

            selector.querySelectorAll('span').forEach(emojiSpan => {
                emojiSpan.addEventListener('click', () => {
                    selector.querySelectorAll('span').forEach(s => s.classList.remove('selected'));
                    emojiSpan.classList.add('selected');
                    targetInput.value = emojiSpan.dataset.value;
                });
            });
        }

        function toggleHabitDetail(habit) {
            const selectElement = document.getElementById(`habito-${habit}`);
            const detailDiv = document.getElementById(`habito-${habit}-detail`);
            if (!selectElement || !detailDiv) return; // Add null check

            if (selectElement.value === 'si') {
                detailDiv.classList.remove('hidden');
            } else {
                detailDiv.classList.add('hidden');
            }
        }

        function toggleHematomaDetails() {
            const selectElement = document.getElementById('hematoma-present');
            const detailsDiv = document.getElementById('hematoma-details');
            if (!selectElement || !detailsDiv) return; // Add null check

            if (selectElement.value === 'si') {
                detailsDiv.classList.remove('hidden');
            } else {
                detailsDiv.classList.add('hidden');
            }
        }

        function toggleHemorrhageDetails() {
            const selectElement = document.getElementById('hemorragia-present');
            const detailsDiv = document.getElementById('hemorragia-details');
            if (!selectElement || !detailsDiv) return; // Add null check

            if (selectElement.value === 'si') {
                detailsDiv.classList.remove('hidden');
            } else {
                detailsDiv.classList.add('hidden');
            }
        }
        
        function toggleOsteoDetail(item) {
            const selectElement = document.getElementById(`osteo-${item}`);
            const detailDiv = document.getElementById(`osteo-${item}-detail`);
            if (!selectElement || !detailDiv) return;

            if (selectElement.value === 'si') {
                detailDiv.classList.remove('hidden');
            } else {
                detailDiv.classList.add('hidden');
            }
        }

        function toggleEndocrineDetail(item) {
            const selectElement = document.getElementById(`endo-${item}`);
            const detailDiv = document.getElementById(`endo-${item}-detail`);
            if (!selectElement || !detailDiv) return;

            if (selectElement.value === 'si') {
                detailDiv.classList.remove('hidden');
            } else {
                detailDiv.classList.add('hidden');
            }
        }

        function checkVitals() {
            const paSistolica = document.getElementById('pa-sistolica').value;
            const paDiastolica = document.getElementById('pa-diastolica').value;
            const fc = document.getElementById('fc').value;
            const fr = document.getElementById('fr').value;
            const temp = document.getElementById('temp').value;
            const spo2 = document.getElementById('spo2').value;
            const glucemia = document.getElementById('glucemia').value;

            const paSistolicaAlert = document.getElementById('pa-sistolica-alert');
            const paDiastolicaAlert = document.getElementById('pa-diastolica-alert');
            const fcAlert = document.getElementById('fc-alert');
            const frAlert = document.getElementById('fr-alert');
            const tempAlert = document.getElementById('temp-alert');
            const spo2Alert = document.getElementById('spo2-alert');
            const glucemiaAlert = document.getElementById('glucemia-alert');
            const spo2CareAlert = document.getElementById('spo2-care-alert');

            // PA Sistólica
            if (paSistolica > 140) {
                paSistolicaAlert.textContent = 'Hipertensión';
                paSistolicaAlert.className = 'red-flag red-flag-icon';
            } else if (paSistolica < 90 && paSistolica) {
                paSistolicaAlert.textContent = 'Hipotensión';
                paSistolicaAlert.className = 'red-flag red-flag-icon';
            } else {
                paSistolicaAlert.textContent = '';
            }
             // PA Diastólica
            if (paDiastolica > 90) {
                paDiastolicaAlert.textContent = 'Hipertensión';
                paDiastolicaAlert.className = 'red-flag red-flag-icon';
            } else if (paDiastolica < 60 && paDiastolica) {
                paDiastolicaAlert.textContent = 'Hipotensión';
                paDiastolicaAlert.className = 'red-flag red-flag-icon';
            } else {
                paDiastolicaAlert.textContent = '';
            }

            // FC
            if (fc > 100) {
                fcAlert.textContent = 'Taquicardia';
                fcAlert.className = 'red-flag red-flag-icon';
            } else if (fc < 60 && fc) {
                fcAlert.textContent = 'Bradicardia';
                fcAlert.className = 'red-flag red-flag-icon';
            } else {
                fcAlert.textContent = '';
            }
             // FR
            if (fr > 20) {
                frAlert.textContent = 'Taquipnea';
                frAlert.className = 'red-flag red-flag-icon';
            } else if (fr < 12 && fr) {
                frAlert.textContent = 'Bradipnea';
                frAlert.className = 'red-flag red-flag-icon';
            } else {
                frAlert.textContent = '';
            }
             // Temp
            if (temp > 38) {
                tempAlert.textContent = 'Fiebre';
                tempAlert.className = 'red-flag red-flag-icon';
            } else if (temp < 35 && temp) {
                tempAlert.textContent = 'Hipotermia';
                tempAlert.className = 'red-flag red-flag-icon';
            } else {
                tempAlert.textContent = '';
            }
            // SpO2 and care
            if (spo2 >= 94 && spo2 <= 100) {
                spo2Alert.textContent = 'Normal';
                spo2Alert.className = 'text-green-600 font-bold';
                spo2CareAlert.textContent = 'Cuidado General: Dar oxígeno según necesidad.';
                spo2CareAlert.classList.remove('red-flag', 'bg-yellow-100', 'border-yellow-300', 'text-yellow-800');
                spo2CareAlert.classList.add('bg-green-100', 'border-green-300', 'text-green-800');
                spo2CareAlert.classList.remove('hidden');
            } else if (spo2 >= 91 && spo2 <= 93) {
                spo2Alert.textContent = 'Hipoxia Leve';
                spo2Alert.className = 'red-flag red-flag-icon';
                spo2CareAlert.textContent = 'Cuidado General: Dar oxígeno según necesidad.';
                spo2CareAlert.classList.remove('hidden');
                spo2CareAlert.classList.remove('bg-green-100', 'border-green-300', 'text-green-800');
                spo2CareAlert.classList.add('bg-yellow-100', 'border-yellow-300', 'text-yellow-800');
            } else if (spo2 >= 86 && spo2 <= 90) {
                spo2Alert.textContent = 'Hipoxia Moderada';
                spo2Alert.className = 'red-flag red-flag-icon';
                spo2CareAlert.textContent = 'Cuidado General: Dar 100% oxígeno. Asistir ventilaciones si es necesario.';
                spo2CareAlert.classList.remove('hidden');
                spo2CareAlert.classList.remove('bg-green-100', 'border-green-300', 'text-green-800', 'bg-yellow-100', 'border-yellow-300', 'text-yellow-800');
                spo2CareAlert.classList.add('red-flag', 'bg-red-100', 'border-red-300', 'text-red-800');
            } else if (spo2 <= 85 && spo2) {
                spo2Alert.textContent = 'Hipoxia Severa';
                spo2Alert.className = 'red-flag red-flag-icon';
                spo2CareAlert.textContent = 'Cuidado General: Dar 100% oxígeno. Asistir ventilaciones si es necesario. Si está indicado, Intubar.';
                spo2CareAlert.classList.remove('hidden');
                spo2CareAlert.classList.remove('bg-green-100', 'border-green-300', 'text-green-800', 'bg-yellow-100', 'border-yellow-300', 'text-yellow-800');
                spo2CareAlert.classList.add('red-flag', 'bg-red-200', 'border-red-400', 'text-red-900');
            } else {
                spo2Alert.textContent = '';
                spo2CareAlert.classList.add('hidden');
            }

            // Glucemia
            if (glucemia > 180) {
                 glucemiaAlert.textContent = 'Hiperglucemia';
                 glucemiaAlert.className = 'red-flag red-flag-icon';
            } else if (glucemia < 70 && glucemia) {
                 glucemiaAlert.textContent = 'Hipoglucemia';
                 glucemiaAlert.className = 'red-flag red-flag-icon';
            } else {
                glucemiaAlert.textContent = '';
            }
        }

        // --- Abdomen 9 Quadrants Logic ---
        const nineQuadrantOrganData = {
            'hipocondrio-derecho': {
                organs: 'Hígado, Vesícula Biliar, Riñón derecho (superior), Flexura hepática del colon.',
                term: 'Hipocondrio derecho'
            },
            'epigastrio': {
                organs: 'Estómago (mayor parte), Páncreas, Duodeno (parte), Lóbulos hepáticos.',
                term: 'Epigastrio'
            },
            'hipocondrio-izquierdo': {
                organs: 'Bazo, Estómago (fondo), Cola del páncreas, Riñón izquierdo (superior), Flexura esplénica del colon.',
                term: 'Hipocondrio izquierdo'
            },
            'flanco-derecho': {
                organs: 'Colon ascendente, Riñón derecho (inferior), Parte del intestino delgado.',
                term: 'Flanco derecho'
            },
            'mesogastrio': {
                organs: 'Intestino delgado (yeyuno e íleon), Ombligo, Aorta abdominal, Vena cava inferior.',
                term: 'Mesogastrio'
            },
            'flanco-izquierdo': {
                organs: 'Colon descendente, Riñón izquierdo (inferior), Parte del intestino delgado.',
                term: 'Flanco izquierdo'
            },
            'fosa-iliaca-derecha': {
                organs: 'Ciego, Apéndice, Íleon terminal, Ovario y trompa de Falopio derecha (mujeres), Cordón espermático derecho (hombres).',
                term: 'Fosa ilíaca derecha'
            },
            'hipogastrio': {
                organs: 'Vejiga, Útero (mujeres), Próstata (hombres), Intestino delgado (parte), Colon sigmoides.',
                term: 'Hipogastrio'
            },
            'fosa-iliaca-izquierda': {
                organs: 'Colon sigmoides, Ovario y trompa de Falopio izquierda (mujeres), Cordón espermático izquierdo (hombres).',
                term: 'Fosa ilíaca izquierda'
            }
        };
        let currentQuadrantTerm = '';

        function showOrgansNineQuadrants(quadrantId) {
            const infoBox = document.getElementById('nine-quadrant-info');
            const organsListSpan = document.getElementById('organs-list');
            const data = nineQuadrantOrganData[quadrantId];

            if (data) {
                organsListSpan.textContent = data.organs;
                currentQuadrantTerm = data.term; // Set current term for pathology explanation
                infoBox.style.display = 'block';
                document.getElementById('organ-pathology-output').textContent = ''; // Clear previous output
                document.getElementById('organ-pathology-output').classList.add('hidden');
            } else {
                infoBox.style.display = 'none';
            }
        }

        async function explainOrganPathologies() {
            const outputDiv = document.getElementById('organ-pathology-output');
            const loadingSpinner = document.getElementById('organ-pathology-loading');
            const term = currentQuadrantTerm; // Use the term of the last clicked quadrant

            if (!term) {
                outputDiv.textContent = 'Por favor, seleccione un cuadrante primero.';
                outputDiv.classList.remove('hidden');
                return;
            }

            outputDiv.textContent = '';
            outputDiv.classList.add('hidden');
            loadingSpinner.classList.remove('hidden');

            try {
                const organs = nineQuadrantOrganData[Object.keys(nineQuadrantOrganData).find(key => nineQuadrantOrganData[key].term === term)].organs;
                let prompt = `Actúa como un médico experimentado. Para la región abdominal de "${term}" que contiene los órganos: ${organs}, describe de forma concisa 2-3 patologías comunes y sus posibles causas, utilizando un lenguaje claro y médico.`;
                
                const apiKey = "";
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`;
                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ contents: [{ role: "user", parts: [{ text: prompt }] }] })
                });

                if (!response.ok) { // Check if response is OK (status 200-299)
                    const errorText = await response.text();
                    throw new Error(`HTTP error! status: ${response.status}, body: ${errorText}`);
                }

                const result = await response.json();
                
                if (result.candidates && result.candidates.length > 0 && result.candidates[0].content && result.candidates[0].content.parts.length > 0) {
                    outputDiv.textContent = result.candidates[0].content.parts[0].text;
                } else {
                    outputDiv.textContent = 'Error: No se pudo generar la explicación de patologías. Respuesta inesperada de la API.';
                }
                outputDiv.classList.remove('hidden');

            } catch (error) {
                console.error('Error al llamar a Gemini API para patologías:', error);
                outputDiv.textContent = `Error al generar la explicación de patologías: ${error.message}`;
                outputDiv.classList.remove('hidden');
            } finally {
                loadingSpinner.classList.add('hidden');
            }
        }

        // --- Auscultation Logic ---
        async function auscultatePoint(pointName) {
            const infoBox = document.getElementById('auscultation-info');
            const pointNameSpan = document.getElementById('auscultation-point-name');
            const explanationOutput = document.getElementById('auscultation-explanation-output');
            const loadingSpinner = document.getElementById('auscultation-loading');

            pointNameSpan.textContent = pointName;
            infoBox.style.display = 'block';
            explanationOutput.textContent = '';
            explanationOutput.classList.add('hidden');
            loadingSpinner.classList.remove('hidden');

            try {
                let prompt = `Para el punto de auscultación "${pointName}", describe los sonidos normales esperados (murmullos, ruidos) y 2-3 patologías o hallazgos anormales comunes que se podrían escuchar en un lenguaje médico claro.`;
                
                const apiKey = "";
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`;
                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ contents: [{ role: "user", parts: [{ text: prompt }] }] })
                });

                if (!response.ok) { // Check if response is OK (status 200-299)
                    const errorText = await response.text();
                    throw new Error(`HTTP error! status: ${response.status}, body: ${errorText}`);
                }

                const result = await response.json();
                
                if (result.candidates && result.candidates.length > 0 && result.candidates[0].content && result.candidates[0].content.parts.length > 0) {
                    explanationOutput.textContent = result.candidates[0].content.parts[0].text;
                } else {
                    explanationOutput.textContent = 'Error: No se pudo generar la explicación de auscultación. Respuesta inesperada de la API.';
                }
                explanationOutput.classList.remove('hidden');

            } catch (error) {
                console.error('Error al llamar a Gemini API para auscultación:', error);
                explanationOutput.textContent = `Error al generar la explicación de auscultación: ${error.message}`;
                explanationOutput.classList.remove('hidden');
            } finally {
                loadingSpinner.classList.add('hidden');
            }
        }

        // --- JVD Differentiation Logic ---
        function toggleJVDDifferentiation() {
            const jvdSelect = document.getElementById('jvd-present');
            const differentiateBtn = document.getElementById('differentiate-jvd-btn');
            const outputDiv = document.getElementById('jvd-differentiation-output');
            
            if (jvdSelect.value === 'si') {
                differentiateBtn.classList.remove('hidden');
            } else {
                differentiateBtn.classList.add('hidden');
                outputDiv.classList.add('hidden');
            }
        }

        async function differentiateJVD() {
            const outputDiv = document.getElementById('jvd-differentiation-output');
            const loadingSpinner = document.getElementById('jvd-loading');

            outputDiv.textContent = '';
            outputDiv.classList.add('hidden');
            loadingSpinner.classList.remove('hidden');

            try {
                let prompt = `Un paciente presenta distensión de venas yugulares (JVD). Explica cómo diferenciar una tamponada cardíaca de un neumotórax a tensión (o hemoneumotórax) basándose en signos y síntomas adicionales clave. Detalla los signos y síntomas específicos para cada patología que ayudarían a la diferenciación.`;
                
                const apiKey = "";
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`;
                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ contents: [{ role: "user", parts: [{ text: prompt }] }] })
                });

                if (!response.ok) { // Check if response is OK (status 200-299)
                    const errorText = await response.text();
                    throw new Error(`HTTP error! status: ${response.status}, body: ${errorText}`);
                }

                const result = await response.json();
                
                if (result.candidates && result.candidates.length > 0 && result.candidates[0].content && result.candidates[0].content.parts.length > 0) {
                    outputDiv.textContent = result.candidates[0].content.parts[0].text;
                } else {
                    outputDiv.textContent = 'Error: No se pudo generar la explicación de diferenciación. Respuesta inesperada de la API.';
                }
                outputDiv.classList.remove('hidden');

            } catch (error) {
                console.error('Error al llamar a Gemini API para JVD:', error);
                outputDiv.textContent = `Error al generar la explicación de diferenciación: ${error.message}`;
                outputDiv.classList.remove('hidden');
            } finally {
                loadingSpinner.classList.add('hidden');
            }
        }

        // --- Mannequin Lesion Marking ---
        let lesions = [];
        function markLesion(event, view) {
            const figure = event.currentTarget;
            const rect = figure.getBoundingClientRect();
            const x = event.clientX - rect.left;
            const y = event.clientY - rect.top;

            const description = prompt(`Describa la lesión en ${view} (X: ${x.toFixed(0)}, Y: ${y.toFixed(0)}):`);
            if (description) {
                const lesion = { x, y, view, description };
                lesions.push(lesion);
                renderLesions();
            }
        }

        function renderLesions() {
            document.querySelectorAll('.anatomical-figure').forEach(figure => {
                figure.querySelectorAll('.lesion-point').forEach(point => point.remove());
            });

            const lesionList = document.getElementById('lesion-list');
            lesionList.innerHTML = '<h4 class="font-semibold mb-2">Lesiones Marcadas:</h4>';

            lesions.forEach(lesion => {
                const figure = document.querySelector(`.anatomical-figure[data-view="${lesion.view}"]`);
                if (figure) {
                    const point = document.createElement('div');
                    point.className = 'lesion-point';
                    point.style.left = `${lesion.x}px`;
                    point.style.top = `${lesion.y}px`;
                    figure.appendChild(point);
                }
                
                const listItem = document.createElement('p');
                listItem.textContent = `• (${lesion.view}) X:${lesion.x.toFixed(0)}, Y:${lesion.y.toFixed(0)}: ${lesion.description}`;
                lesionList.appendChild(listItem);
            });
        }
        
        // --- Medication Management ---
        function addMedicationRow() {
            const tableBody = document.getElementById('medication-table-body');
            const newRow = document.createElement('div');
            newRow.className = 'grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4 medication-row border-b pb-4 mb-4';
            newRow.innerHTML = `
                <div class="input-group"><label class="label">Medicamento</label><input type="text" class="input-field medication-name"></div>
                <div class="input-group"><label class="label">Dosis</label><input type="text" class="input-field medication-dose"></div>
                <div class="input-group"><label class="label">Vía</label><input type="text" class="input-field medication-via"></div>
                <div class="input-group"><label class="label">Frecuencia</label><input type="text" class="input-field medication-frequency" placeholder="ej. Cada 8h, QD, BID"></div>
                <div class="input-group"><label class="label">Última Dosis (HH:MM)</label><input type="time" class="input-field medication-last-time"></div>
                <div class="input-group"><label class="label">Próxima Dosis</label><input type="text" class="input-field medication-next-time" readonly></div>
                <button onclick="getDrugDetails(this)" class="px-3 py-1 bg-blue-600 text-white rounded-lg hover:bg-blue-700 text-sm md:col-span-2 mt-4 md:mt-0">Detalles del Medicamento ✨</button>
                <div class="col-span-full mt-2 hidden drug-details-output bg-gray-100 p-2 rounded text-sm relative">
                    <div class="spinner absolute top-1 right-1 hidden"></div>
                    <p class="text-xs text-gray-500 mt-2">La información generada por IA es para fines informativos y de apoyo, NO sustituye el juicio clínico.</p>
                </div>
            `;
            tableBody.appendChild(newRow);
            // Re-attach event listeners for new inputs
            newRow.querySelector('.medication-name').addEventListener('input', calculateNextMedTime);
            newRow.querySelector('.medication-frequency').addEventListener('input', calculateNextMedTime);
            newRow.querySelector('.medication-last-time').addEventListener('input', calculateNextMedTime);
        }

        function calculateNextMedTime() {
            const rows = document.querySelectorAll('.medication-row');
            rows.forEach(row => {
                const lastTimeInput = row.querySelector('.medication-last-time');
                const frequencyInput = row.querySelector('.medication-frequency');
                const nextTimeInput = row.querySelector('.medication-next-time');

                const lastTime = lastTimeInput.value;
                const frequency = frequencyInput.value;

                if (lastTime && frequency) {
                    const [hours, minutes] = lastTime.split(':').map(Number);
                    let nextTime = new Date();
                    nextTime.setHours(hours, minutes, 0, 0);

                    // Basic frequency parsing (e.g., "Cada 8h", "QD", "BID", "TID", "QID")
                    if (frequency.toLowerCase().includes('cada')) {
                        const freqNum = parseInt(frequency.toLowerCase().replace('cada ', '').replace('h', ''));
                        if (!isNaN(freqNum)) {
                            nextTime.setHours(nextTime.getHours() + freqNum);
                        }
                    } else if (frequency.toLowerCase() === 'qd') {
                        nextTime.setHours(nextTime.getHours() + 24);
                    } else if (frequency.toLowerCase() === 'bid') {
                        nextTime.setHours(nextTime.getHours() + 12);
                    } else if (frequency.toLowerCase() === 'tid') {
                        nextTime.setHours(nextTime.getHours() + 8);
                    } else if (frequency.toLowerCase() === 'qid') {
                        nextTime.setHours(nextTime.getHours() + 6);
                    }
                    
                    nextTimeInput.value = nextTime.toLocaleTimeString('es-DO', { hour: '2-digit', minute: '2-digit' });

                } else {
                    nextTimeInput.value = '';
                }
            });
        }

        async function getDrugDetails(buttonElement) {
            const row = buttonElement.closest('.medication-row');
            const medicationName = row.querySelector('.medication-name').value;
            const outputDiv = row.querySelector('.drug-details-output');
            const loadingSpinner = outputDiv.querySelector('.spinner');

            if (!medicationName) {
                outputDiv.textContent = 'Por favor, ingrese el nombre del medicamento.';
                outputDiv.classList.remove('hidden');
                return;
            }

            outputDiv.textContent = '';
            outputDiv.classList.add('hidden');
            loadingSpinner.classList.remove('hidden');

            try {
                let prompt = `Actúa como un farmacéutico. Proporciona una descripción concisa de los usos principales, efectos secundarios comunes y precauciones importantes (como riesgo de hipotensión o interacciones clave) del medicamento "${medicationName}". Si es un medicamento con una administración particular como un nebulizador o sublingual con intervalo, menciónalo.`;
                
                const apiKey = "";
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`;
                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ contents: [{ role: "user", parts: [{ text: prompt }] }] })
                });

                if (!response.ok) { // Check if response is OK (status 200-299)
                    const errorText = await response.text();
                    throw new Error(`HTTP error! status: ${response.status}, body: ${errorText}`);
                }

                const result = await response.json();
                
                if (result.candidates && result.candidates.length > 0 && result.candidates[0].content && result.candidates[0].content.parts && result.candidates[0].content.parts.length > 0) {
                    outputDiv.textContent = result.candidates[0].content.parts[0].text;
                } else {
                    outputDiv.textContent = 'Error: No se pudo obtener detalles del medicamento. Respuesta inesperada de la API.';
                }
                outputDiv.classList.remove('hidden');

            } catch (error) {
                console.error('Error al llamar a Gemini API para detalles de medicamento:', error);
                outputDiv.textContent = `Error al obtener detalles del medicamento: ${error.message}`;
                outputDiv.classList.remove('hidden');
            } finally {
                loadingSpinner.classList.add('hidden');
            }
        }

        async function generateDifferentialDiagnosis() {
            const symptomsInput = document.getElementById('symptoms-for-diagnosis');
            const diagnosisOutput = document.getElementById('ai-diagnosis-output');
            const loadingSpinner = document.getElementById('diagnosis-loading');
            const generateBtn = document.getElementById('generate-diagnosis-btn');

            const symptoms = symptomsInput.value.trim();
            if (!symptoms) {
                diagnosisOutput.textContent = 'Por favor, ingrese síntomas y hallazgos para generar diagnósticos.';
                return;
            }

            diagnosisOutput.textContent = '';
            loadingSpinner.classList.remove('hidden');
            generateBtn.disabled = true;

            try {
                let prompt = `Actúa como un médico experimentado. Basado en los siguientes síntomas y hallazgos de un paciente, genera una lista de 3 a 5 posibles diagnósticos diferenciales. Para cada diagnóstico, menciona brevemente uno o dos puntos clave que lo apoyen y uno o dos que lo contradigan (si aplica).

Síntomas y Hallazgos: ${symptoms}

Diagnósticos Diferenciales:`;
                
                const apiKey = "";
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`;
                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ contents: [{ role: "user", parts: [{ text: prompt }] }] })
                });

                if (!response.ok) {
                    const errorText = await response.text();
                    throw new Error(`HTTP error! status: ${response.status}, body: ${errorText}`);
                }

                const result = await response.json();
                
                if (result.candidates && result.candidates.length > 0 && result.candidates[0].content && result.candidates[0].content.parts.length > 0) {
                    diagnosisOutput.textContent = result.candidates[0].content.parts[0].text;
                } else {
                    diagnosisOutput.textContent = 'Error: No se pudo generar el diagnóstico diferencial. Respuesta inesperada de la API.';
                }

            } catch (error) {
                console.error('Error al llamar a Gemini API para diagnóstico diferencial:', error);
                diagnosisOutput.textContent = `Error al generar el diagnóstico diferencial: ${error.message}`;
            } finally {
                loadingSpinner.classList.add('hidden');
                generateBtn.disabled = false;
            }
        }

        async function refineDiagnosis() {
            const diagnosisInput = document.getElementById('diagnosis-refiner-input');
            const refinerOutput = document.getElementById('ai-refiner-output');
            const loadingSpinner = document.getElementById('refiner-loading');
            const refineBtn = document.getElementById('refine-diagnosis-btn');

            const diagnosis = diagnosisInput.value.trim();
            if (!diagnosis) {
                refinerOutput.textContent = 'Por favor, ingrese un diagnóstico sospechado para refinar.';
                return;
            }

            refinerOutput.textContent = '';
            loadingSpinner.classList.remove('hidden');
            refineBtn.disabled = true;

            try {
                let prompt = `Actúa como un experto médico. Para el diagnóstico sospechado de "${diagnosis}", describe concisamente 3-4 características clínicas clave o pruebas diagnósticas que son cruciales para confirmarlo o descartarlo, y cómo se diferenciaría de condiciones similares. Enfócate en los puntos diferenciadores.`;
                
                const apiKey = "";
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`;
                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ contents: [{ role: "user", parts: [{ text: prompt }] }] })
                });

                if (!response.ok) {
                    const errorText = await response.text();
                    throw new Error(`HTTP error! status: ${response.status}, body: ${errorText}`);
                }

                const result = await response.json();
                
                if (result.candidates && result.candidates.length > 0 && result.candidates[0].content && result.candidates[0].content.parts.length > 0) {
                    refinerOutput.textContent = result.candidates[0].content.parts[0].text;
                } else {
                    refinerOutput.textContent = 'Error: No se pudo refinar el diagnóstico. Respuesta inesperada de la API.';
                }

            } catch (error) {
                console.error('Error al llamar a Gemini API para refinador de diagnóstico:', error);
                refinerOutput.textContent = `Error al refinar el diagnóstico: ${error.message}`;
            } finally {
                loadingSpinner.classList.add('hidden');
                refineBtn.disabled = false;
            }
        }


        function calculateGCS() {
            const ocular = parseInt(document.getElementById('gcs-ocular').value);
            const verbal = parseInt(document.getElementById('gcs-verbal').value);
            const motor = parseInt(document.getElementById('gcs-motor').value);
            const total = ocular + verbal + motor;
            
            document.getElementById('gcs-total').textContent = total;

            const alertEl = document.getElementById('gcs-alert');
            if (total < 8) {
                alertEl.textContent = 'RIESGO DE VÍA AÉREA - CONSIDERAR INTUBACIÓN';
                alertEl.className = 'red-flag red-flag-icon font-bold';
            } else {
                alertEl.textContent = '';
                alertEl.className = 'font-bold';
            }
        }

        function calculateParkland() {
            const weight = parseFloat(document.getElementById('burn-weight').value);
            const bsa = parseFloat(document.getElementById('burn-bsa').value);
            const resultsDiv = document.getElementById('parkland-results');

            if (weight > 0 && bsa > 0) {
                const totalVolume = 4 * weight * bsa;
                const first8h = totalVolume / 2;
                const next16h = totalVolume / 2;

                document.getElementById('parkland-total').textContent = totalVolume.toFixed(2);
                document.getElementById('parkland-first-8h').textContent = first8h.toFixed(2);
                document.getElementById('parkland-next-16h').textContent = next16h.toFixed(2);
                resultsDiv.style.display = 'block';

                const alertEl = document.getElementById('burn-alert');
                if (bsa > 20) {
                     alertEl.textContent = 'QUEMADURA MAYOR';
                     alertEl.className = 'red-flag red-flag-icon';
                } else {
                    alertEl.textContent = '';
                }
            } else {
                resultsDiv.style.display = 'none';
            }
        }

        function calculateDose() {
            const desiredDose = parseFloat(document.getElementById('calc-dosis-deseada').value);
            const weight = parseFloat(document.getElementById('calc-dosis-peso').value);
            const concentration = parseFloat(document.getElementById('calc-dosis-concentracion').value);

            if (desiredDose > 0 && weight > 0 && concentration > 0) {
                const doseToAdminister = (desiredDose * weight) / concentration;
                document.getElementById('result-dosis').textContent = doseToAdminister.toFixed(2);
            } else {
                document.getElementById('result-dosis').textContent = '0';
            }
        }

        function calculateDripRate() {
            const volume = parseFloat(document.getElementById('calc-goteo-volumen').value);
            const time = parseFloat(document.getElementById('calc-goteo-tiempo').value);
            const factor = parseFloat(document.getElementById('calc-goteo-factor').value);

            if (volume > 0 && time > 0 && factor > 0) {
                const dripRate = (volume * factor) / (time * 60);
                document.getElementById('result-goteo').textContent = dripRate.toFixed(2);
            } else {
                document.getElementById('result-goteo').textContent = '0';
            }
        }

        function calculateDehydration() {
            const degree = parseFloat(document.getElementById('dehydration-degree').value);
            const weight = parseFloat(document.getElementById('dehydration-weight').value);
            
            if (degree > 0 && weight > 0) {
                const deficit = weight * degree * 10;
                let basalNeeds = 0;
                if (weight <= 10) {
                    basalNeeds = weight * 100;
                } else if (weight <= 20) {
                    basalNeeds = (10 * 100) + ((weight - 10) * 50);
                } else {
                    basalNeeds = (10 * 100) + (10 * 50) + ((weight - 20) * 20);
                }
                const total24h = deficit + basalNeeds;

                document.getElementById('dehydration-deficit').textContent = deficit.toFixed(2);
                document.getElementById('dehydration-basal').textContent = basalNeeds.toFixed(2);
                document.getElementById('dehydration-total-24h').textContent = total24h.toFixed(2);
            } else {
                document.getElementById('dehydration-deficit').textContent = '0';
                document.getElementById('dehydration-basal').textContent = '0';
                document.getElementById('dehydration-total-24h').textContent = '0';
            }
        }

        async function generateSummary() {
            const summaryOutput = document.getElementById('ai-summary-output');
            const loadingSpinner = document.getElementById('summary-loading');
            const generateBtn = document.getElementById('generate-summary-btn');

            summaryOutput.textContent = ''; // Clear previous summary
            loadingSpinner.classList.remove('hidden');
            generateBtn.disabled = true;

            try {
                // Collect data from various sections (simplified for demo)
                const patientName = document.getElementById('patient-name').value || 'no especificado';
                const patientAge = document.getElementById('patient-age').value || 'no especificada';
                const patientSex = document.getElementById('patient-sex').value || 'no especificado';
                
                const mainSymptom = document.getElementById('main-symptom').value || 'no especificado';
                const patientIdeas = document.getElementById('patient-ideas').value || 'no especificado';
                const patientConcerns = document.getElementById('patient-concerns').value || 'no especificado';
                const functionalImpact = document.getElementById('functional-impact').value || 'no especificado';

                const paSistolica = document.getElementById('pa-sistolica').value || 'N/A';
                const paDiastolica = document.getElementById('pa-diastolica').value || 'N/A';
                const fc = document.getElementById('fc').value || 'N/A';
                const fr = document.getElementById('fr').value || 'N/A';
                const temp = document.getElementById('temp').value || 'N/A';
                const spo2 = document.getElementById('spo2').value || 'N/A';
                const glucemia = document.getElementById('glucemia').value || 'N/A';

                const heaOnset = document.getElementById('hea-onset').value || 'no especificada';
                const heaType = document.getElementById('hea-type').value || 'no especificado';
                const heaIntensity = document.getElementById('hea-intensity').value || 'no especificada';
                const heaLocation = document.getElementById('hea-location').value || 'no especificada';
                const heaAccompanying = document.getElementById('hea-accompanying').value || 'no especificados';
                const heaPrevTreatments = document.getElementById('hea-prev-treatments').value || 'ninguno';

                let prompt = `Genera un resumen clínico conciso y profesional de la siguiente historia clínica de un paciente. Organiza la información en secciones claras como Datos del Paciente, Motivo de Consulta (incluyendo ICE), Historia de Enfermedad Actual, y Signos Vitales. Destaca solo la información relevante y omite los campos vacíos.

**Datos del Paciente:**
Nombre: ${patientName}
Edad: ${patientAge}
Sexo: ${patientSex}

**Motivo de Consulta:**
Síntoma Principal: ${mainSymptom}
Ideas del paciente: ${patientIdeas}
Preocupaciones del paciente: ${patientConcerns}
Impacto funcional: ${functionalImpact}

**Historia de Enfermedad Actual:**
Aparición: ${heaOnset}
Tipo: ${heaType}
Intensidad: ${heaIntensity}
Localización: ${heaLocation}
Fenómenos acompañantes: ${heaAccompanying}
Tratamientos previos: ${heaPrevTreatments}

**Signos Vitales:**
PA: ${paSistolica}/${paDiastolica} mmHg
FC: ${fc} lpm
FR: ${fr} rpm
Temperatura: ${temp} °C
SpO2: ${spo2} %
Glucemia: ${glucemia} mg/dL

Resumen:
`;
                
                let chatHistory = [];
                chatHistory.push({ role: "user", parts: [{ text: prompt }] });
                const payload = { contents: chatHistory };
                const apiKey = ""; 
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`;

                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });

                if (!response.ok) { // Check if response is OK (status 200-299)
                    const errorText = await response.text();
                    throw new Error(`HTTP error! status: ${response.status}, body: ${errorText}`);
                }
                
                const result = await response.json();
                
                if (result.candidates && result.candidates.length > 0 &&
                    result.candidates[0].content && result.candidates[0].content.parts &&
                    result.candidates[0].content.parts.length > 0) {
                    const text = result.candidates[0].content.parts[0].text;
                    summaryOutput.textContent = text;
                } else {
                    summaryOutput.textContent = 'Error: No se pudo generar el resumen. Respuesta inesperada de la API.';
                }

            } catch (error) {
                console.error('Error al llamar a Gemini API:', error);
                summaryOutput.textContent = `Error al generar el resumen: ${error.message}`;
            } finally {
                loadingSpinner.classList.add('hidden');
                generateBtn.disabled = false;
            }
        }

        async function explainMedicalTerm() {
            const termInput = document.getElementById('medical-term-input');
            const explanationOutput = document.getElementById('ai-explanation-output');
            const loadingSpinner = document.getElementById('explanation-loading');
            const explainBtn = document.getElementById('explain-term-btn');

            const term = termInput.value.trim();
            if (!term) {
                explanationOutput.textContent = 'Por favor, ingrese un término para explicar.';
                return;
            }

            explanationOutput.textContent = ''; // Clear previous explanation
            loadingSpinner.classList.remove('hidden');
            explainBtn.disabled = true;

            try {
                let prompt = `Explica el siguiente término o concepto médico en un lenguaje sencillo y fácil de entender para un paciente, sin usar jerga compleja. Concéntrate en la esencia y el impacto práctico para el paciente.
                
                Término: ${term}
                
                Explicación:`;
                
                let chatHistory = [];
                chatHistory.push({ role: "user", parts: [{ text: prompt }] });
                const payload = { contents: chatHistory };
                const apiKey = ""; 
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`;

                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });

                if (!response.ok) { // Check if response is OK (status 200-299)
                    const errorText = await response.text();
                    throw new Error(`HTTP error! status: ${response.status}, body: ${errorText}`);
                }
                
                const result = await response.json();
                
                if (result.candidates && result.candidates.length > 0 &&
                    result.candidates[0].content && result.candidates[0].content.parts &&
                    result.candidates[0].content.parts.length > 0) {
                    const text = result.candidates[0].content.parts[0].text;
                    explanationOutput.textContent = text;
                } else {
                    explanationOutput.textContent = 'Error: No se pudo generar la explicación. Respuesta inesperada de la API.';
                }

            } catch (error) {
                console.error('Error al llamar a Gemini API:', error);
                explanationOutput.textContent = `Error al generar la explicación: ${error.message}`;
            } finally {
                loadingSpinner.classList.add('hidden');
                explainBtn.disabled = false;
            }
        }

        async function generatePatientEducation() {
            const educationInput = document.getElementById('patient-education-input');
            const educationOutput = document.getElementById('ai-patient-education-output');
            const loadingSpinner = document.getElementById('patient-education-loading');
            const generateBtn = document.getElementById('generate-patient-education-btn');

            const topic = educationInput.value.trim();
            if (!topic) {
                educationOutput.textContent = 'Por favor, ingrese una condición o tratamiento.';
                return;
            }

            educationOutput.textContent = '';
            loadingSpinner.classList.remove('hidden');
            generateBtn.disabled = true;

            try {
                let prompt = `Genera un material educativo conciso y fácil de entender para un paciente sobre "${topic}". Utiliza un lenguaje sencillo, evita la jerga médica y concéntrate en lo que el paciente necesita saber sobre su condición/tratamiento, cómo manejarla en casa y cuándo buscar ayuda. Incluye 3-4 puntos clave.`;
                
                const apiKey = "";
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`;
                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ contents: [{ role: "user", parts: [{ text: prompt }] }] })
                });

                if (!response.ok) {
                    const errorText = await response.text();
                    throw new Error(`HTTP error! status: ${response.status}, body: ${errorText}`);
                }

                const result = await response.json();
                
                if (result.candidates && result.candidates.length > 0 && result.candidates[0].content && result.candidates[0].content.parts && result.candidates[0].content.parts.length > 0) {
                    const text = result.candidates[0].content.parts[0].text;
                    educationOutput.textContent = text;
                } else {
                    educationOutput.textContent = 'Error: No se pudo generar el material educativo. Respuesta inesperada de la API.';
                }

            } catch (error) {
                console.error('Error al llamar a Gemini API para material educativo:', error);
                educationOutput.textContent = `Error al generar el material educativo: ${error.message}`;
            } finally {
                loadingSpinner.classList.add('hidden');
                generateBtn.disabled = false;
            }
        }

        async function generateFollowUpQuestions() {
            const symptomInput = document.getElementById('follow-up-symptom-input');
            const questionsOutput = document.getElementById('ai-followup-output');
            const loadingSpinner = document.getElementById('followup-loading');
            const generateBtn = document.getElementById('generate-followup-btn');

            const symptom = symptomInput.value.trim();
            if (!symptom) {
                questionsOutput.textContent = 'Por favor, ingrese un síntoma o hallazgo para generar preguntas.';
                return;
            }

            questionsOutput.textContent = '';
            loadingSpinner.classList.remove('hidden');
            generateBtn.disabled = true;

            try {
                let prompt = `Actúa como un médico experimentado. Basado en el siguiente síntoma o hallazgo inicial de un paciente, genera una lista de 3 a 5 preguntas de seguimiento clave que un clínico debería hacer para una anamnesis más completa.

Síntoma/Hallazgo inicial: ${symptom}

Preguntas de seguimiento:`;
                
                const apiKey = "";
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`;
                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ contents: [{ role: "user", parts: [{ text: prompt }] }] })
                });

                if (!response.ok) {
                    const errorText = await response.text();
                    throw new Error(`HTTP error! status: ${response.status}, body: ${errorText}`);
                }

                const result = await response.json();
                
                if (result.candidates && result.candidates.length > 0 && result.candidates[0].content && result.candidates[0].content.parts && result.candidates[0].content.parts.length > 0) {
                    questionsOutput.textContent = result.candidates[0].content.parts[0].text;
                } else {
                    questionsOutput.textContent = 'Error: No se pudo generar las preguntas. Respuesta inesperada de la API.';
                }

            } catch (error) {
                console.error('Error al llamar a Gemini API para preguntas de seguimiento:', error);
                questionsOutput.textContent = `Error al generar las preguntas: ${error.message}`;
            } finally {
                loadingSpinner.classList.add('hidden');
                generateBtn.disabled = false;
            }
        }
    </script>
</body>
</html>