<script setup lang="ts">
import { ref, onMounted, onUnmounted, nextTick } from 'vue'
import L from 'leaflet'
import 'leaflet/dist/leaflet.css'
import { 
  ArrowLeft, 
  ArrowRight, 
  FileCheck2, 
  MessageCircle, 
  X, 
  Download, 
  Info, 
  Calendar, 
  Linkedin, 
  Instagram, 
  Facebook, 
  CheckCircle2, 
  Clock, 
  FileSpreadsheet,
  MapPin,
  Phone,
  Navigation,
  Compass,
  Microscope
} from 'lucide-vue-next'

// Definición de tipos para los slides
interface Slide {
  id: number
  category: string
  title: string
  subtitle: string
  description: string
  image: string
  badgeText: string
  statNumber: string
  statLabel: string
  secondaryStatNumber: string
  secondaryStatLabel: string
  accentColor: string
}

// Especialistas
interface Specialist {
  id: number
  name: string
  role: string
  specialtyDetail: string
  colegiatura: string
  image: string
}

// Estructura de Sedes con Mapa interactivo fluido
interface Branch {
  id: number
  name: string
  district: string
  address: string
  reference: string
  hours: string
  fastingHours: string
  phone: string
  whatsapp: string
  lat: number
  lng: number
  mapsDirectionUrl: string
  tag: string
}

const branches: Branch[] = [
  {
    id: 1,
    name: 'Sede Central San Isidro',
    district: 'San Isidro',
    address: 'Av. Javier Prado Este 2450, Centro Médico El Prado',
    reference: 'Frente a la Clínica Internacional / Estacionamiento privado',
    hours: 'Lun a Vie: 6:30 AM - 6:30 PM | Sáb: 7:00 AM - 1:30 PM',
    fastingHours: 'Toma en ayunas hasta las 10:30 AM',
    phone: '(01) 555-0100',
    whatsapp: '51999999991',
    lat: -12.08985,
    lng: -77.01255,
    mapsDirectionUrl: 'https://www.google.com/maps/dir/?api=1&destination=-12.08985,-77.01255',
    tag: 'Laboratorio Central & Microbiología'
  },
  {
    id: 2,
    name: 'Sede Miraflores Kennedy',
    district: 'Miraflores',
    address: 'Av. José Pardo 810, Oficina 201',
    reference: 'A 2 cuadras del Parque Kennedy / Óvalo Miraflores',
    hours: 'Lun a Vie: 6:30 AM - 6:00 PM | Sáb: 7:00 AM - 1:00 PM',
    fastingHours: 'Toma en ayunas hasta las 10:00 AM',
    phone: '(01) 555-0200',
    whatsapp: '51999999992',
    lat: -12.11974,
    lng: -77.03456,
    mapsDirectionUrl: 'https://www.google.com/maps/dir/?api=1&destination=-12.11974,-77.03456',
    tag: 'Toma Rápida & Toma Pediátrica'
  },
  {
    id: 3,
    name: 'Sede Surco Chacarilla',
    district: 'Santiago de Surco',
    address: 'Av. Primavera 1420, Módulo de Especialidades',
    reference: 'Cruce con Av. Velasco Astete / Zona Financiera',
    hours: 'Lun a Vie: 7:00 AM - 6:00 PM | Sáb: 7:00 AM - 2:00 PM',
    fastingHours: 'Toma en ayunas hasta las 10:30 AM',
    phone: '(01) 555-0300',
    whatsapp: '51999999993',
    lat: -12.10825,
    lng: -76.98567,
    mapsDirectionUrl: 'https://www.google.com/maps/dir/?api=1&destination=-12.10825,-76.98567',
    tag: 'Chequeos Ejecutivos & Preventivos'
  },
  {
    id: 4,
    name: 'Sede Los Olivos Norte',
    district: 'Los Olivos',
    address: 'Av. Carlos Izaguirre 980, Edificio Médico Norte',
    reference: 'A media cuadra de la Municipalidad de Los Olivos',
    hours: 'Lun a Vie: 6:30 AM - 6:30 PM | Sáb: 7:00 AM - 1:30 PM',
    fastingHours: 'Toma en ayunas hasta las 11:00 AM',
    phone: '(01) 555-0400',
    whatsapp: '51999999994',
    lat: -11.99285,
    lng: -77.07085,
    mapsDirectionUrl: 'https://www.google.com/maps/dir/?api=1&destination=-11.99285,-77.07085',
    tag: 'Atención Continua & Urgencias'
  }
]

// Estado reactivo: Sede seleccionada para el mapa
const selectedBranch = ref<Branch>(branches[0])
const mapContainer = ref<HTMLElement | null>(null)
let leafletMap: L.Map | null = null
let branchMarkers: { [key: number]: L.Marker } = {}

const selectBranch = (branch: Branch) => {
  selectedBranch.value = branch
  if (leafletMap) {
    leafletMap.flyTo([branch.lat, branch.lng], 16, {
      duration: 1.6,
      easeLinearity: 0.25
    })
    const marker = branchMarkers[branch.id]
    if (marker) {
      marker.openPopup()
    }
  }
}

const initMap = () => {
  if (!mapContainer.value) return
  
  // Inicializar Leaflet centrado en la primera sede
  leafletMap = L.map(mapContainer.value, {
    center: [selectedBranch.value.lat, selectedBranch.value.lng],
    zoom: 16,
    zoomControl: false,
    attributionControl: false
  })

  // Capa de mapa libre y pública de OpenStreetMap (sin API Key)
  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    maxZoom: 19,
    subdomains: ['a', 'b', 'c']
  }).addTo(leafletMap)

  // Crear iconos con el logo de Biomedilab
  branches.forEach(b => {
    const customIcon = L.divIcon({
      className: 'custom-biomedilab-pin',
      html: `
        <div style="position: relative; display: flex; flex-direction: column; align-items: center; cursor: pointer;">
          <div style="width: 44px; height: 44px; border-radius: 14px; background: #001D69; border: 2.5px solid #00B4FF; box-shadow: 0 8px 20px rgba(0,180,255,0.6); display: flex; align-items: center; justify-content: center; padding: 4px;">
            <img src="/logo.png" style="width: 100%; height: 100%; object-fit: contain;" />
          </div>
          <div style="width: 0; height: 0; border-left: 6px solid transparent; border-right: 6px solid transparent; border-top: 7px solid #001D69;"></div>
          <div style="margin-top: 4px; background: #001D69; color: white; font-weight: 800; font-size: 10px; padding: 2px 8px; border-radius: 10px; border: 1px solid rgba(255,255,255,0.2); white-space: nowrap; box-shadow: 0 4px 10px rgba(0,0,0,0.3);">
            ${b.district}
          </div>
        </div>
      `,
      iconSize: [44, 60],
      iconAnchor: [22, 54]
    })

    const marker = L.marker([b.lat, b.lng], { icon: customIcon }).addTo(leafletMap!)
    marker.on('click', () => {
      selectBranch(b)
    })
    branchMarkers[b.id] = marker
  })
}

// Estructura de la Hoja de Resultados
interface Biomarker {
  group?: string
  name: string
  result: string
  unit: string
  refRange: string
  status: 'normal' | 'alerta' | 'optimo'
  isHighlighted?: boolean
  description?: string // Explicación médica accesible del analito
}

interface ServiceItem {
  id: number
  name: string
  category: string
  deliveryTime: string
  fastingHours: string
  sampleType: string
  shortDesc: string
  methodology: string
  biomarkers: Biomarker[]
  clinicalNote: string
}

const specialists: Specialist[] = [
  {
    id: 1,
    name: 'Dra. Valeria Mendoza',
    role: 'Patología Clínica & Hematología',
    specialtyDetail: 'Especialista en frotis sanguíneo, diagnóstico de anemias complejas y citología.',
    colegiatura: 'CMP: 58492 • RNE: 32410',
    image: '/doctor_1.jpg'
  },
  {
    id: 2,
    name: 'Dr. Carlos Benavides',
    role: 'Microbiología & Biología Molecular',
    specialtyDetail: 'Monitoreo de cultivos automatizados, PCR en tiempo real y pruebas genómicas.',
    colegiatura: 'CMP: 61204 • RNE: 34190',
    image: '/doctor_2.jpg'
  },
  {
    id: 3,
    name: 'Dra. Gabriela Salazar',
    role: 'Inmunología & Endocrinología',
    specialtyDetail: 'Diagnóstico de enfermedades autoinmunes, perfiles tiroideos y hormonales.',
    colegiatura: 'CMP: 54930 • RNE: 29815',
    image: '/doctor_3.jpg'
  },
  {
    id: 4,
    name: 'Dr. Kenneth Fong',
    role: 'Bioquímica Clínica & Toxicología',
    specialtyDetail: 'Supervisión de control de calidad analítico de equipos de última generación.',
    colegiatura: 'CMP: 63821 • RNE: 36720',
    image: '/doctor_4.jpg'
  }
]

const hoveredSpecialist = ref<number>(2)

// LISTADO DE SERVICIOS
const servicesList: ServiceItem[] = [
  {
    id: 1,
    name: 'HEMOGRAMA COMPLETO',
    category: 'Hematología Automatizada',
    deliveryTime: '3 a 4 Horas',
    fastingHours: '4 Horas (Ligero)',
    sampleType: 'Sangre Total (Tubo EDTA Lila)',
    shortDesc: 'Recuento cuantitativo automatizado de glóbulos rojos, glóbulos blancos, hemoglobina, hematocrito y fórmula leucocitaria.',
    methodology: 'Citometría de Flujo & Impedancia Eléctrica 5 Diferenciales',
    biomarkers: [
      // SERIE ROJA
      { group: 'SERIE ROJA', name: 'HEMATIES', result: '5.12', unit: 'mill/mm3', refRange: '4.5 - 5.9', status: 'optimo', description: 'Glóbulos rojos responsables de transportar el oxígeno desde los pulmones a todos los tejidos del cuerpo.' },
      { group: 'SERIE ROJA', name: 'HEMOGLOBINA', result: '14.3', unit: 'gr/dl', refRange: '14 - 18', status: 'optimo', description: 'Proteína rica en hierro dentro de los hematíes. Es el parámetro principal para descartar o diagnosticar anemia.' },
      { group: 'SERIE ROJA', name: 'HEMATOCRITO', result: '43.7', unit: '%', refRange: '36.2 - 52', status: 'normal', description: 'Porcentaje del volumen total de la sangre ocupado por los glóbulos rojos.' },
      { group: 'SERIE ROJA', name: 'VOLUMEN CORPUSCULAR MEDIO', result: '85.4', unit: 'fL', refRange: '80 - 96', status: 'normal', description: 'Tamaño promedio individual de los glóbulos rojos (microcítico, normocítico o macrocítico).' },
      { group: 'SERIE ROJA', name: 'HEMOGLOBINA CORPUSCULAR MEDIA', result: '27.9', unit: 'pgr', refRange: '28.2 - 33', status: 'alerta', isHighlighted: true, description: 'Cantidad promedio de hemoglobina contenida en cada glóbulo rojo.' },
      { group: 'SERIE ROJA', name: 'CONC. HEMOGL. CORP. MEDIA', result: '32.7', unit: '%', refRange: '30 - 36', status: 'normal', description: 'Concentración de hemoglobina en relación con el volumen del hematíe.' },
      { group: 'SERIE ROJA', name: 'RDW', result: '12.6', unit: '%', refRange: '12 - 16', status: 'normal', description: 'Amplitud de distribución eritrocitaria; evalúa la variación en el tamaño de los glóbulos rojos.' },
      // SERIE BLANCA
      { group: 'SERIE BLANCA', name: 'LEUCOCITOS', result: '10.6', unit: 'x mil/uL', refRange: '4.5 - 11.3', status: 'normal', description: 'Glóbulos blancos totales. Células de defensa inmune que combaten infecciones bacterianas, virales e inflamación.' },
      { group: 'SERIE BLANCA', name: 'NEUTROFILOS %', result: '53.3', unit: '%', refRange: '36 - 77', status: 'normal', description: 'Primera línea de defensa celular contra infecciones bacterianas agudas.' },
      { group: 'SERIE BLANCA', name: 'LINFOCITOS %', result: '31.6', unit: '%', refRange: '20 - 40', status: 'normal', description: 'Células inmunes productoras de anticuerpos y defensa especializada contra virus.' },
      { group: 'SERIE BLANCA', name: 'MONOCITOS %', result: '8.1', unit: '%', refRange: '2 - 9.5', status: 'normal', description: 'Fagocitos que limpian restos celulares y participan en la inmunidad crónica.' },
      { group: 'SERIE BLANCA', name: 'EOSINOFILOS %', result: '6.4', unit: '%', refRange: '0 - 4.4', status: 'alerta', isHighlighted: true, description: 'Glóbulos blancos asociados a procesos alérgicos (asma, rinitis) o respuesta antiparasitaria.' },
      { group: 'SERIE BLANCA', name: 'BASOFILOS %', result: '0.6', unit: '%', refRange: '0 - 1', status: 'normal', description: 'Células que liberan histamina durante respuestas inmunes e inflamatorias.' },
      { group: 'SERIE BLANCA', name: 'NEUTROFILOS', result: '5.6', unit: 'x mil/uL', refRange: '1.6 - 8.7', status: 'normal', description: 'Recuento absoluto de neutrófilos en sangre periférica.' },
      { group: 'SERIE BLANCA', name: 'LINFOCITOS', result: '3.4', unit: 'x mil/uL', refRange: '0.9 - 4.3', status: 'normal', description: 'Recuento absoluto de linfocitos protectores en sangre periférica.' },
      { group: 'SERIE BLANCA', name: 'MONOCITOS', result: '0.9', unit: 'x mil/uL', refRange: '0.2 - 1', status: 'normal', description: 'Recuento absoluto de monocitos en sangre.' },
      { group: 'SERIE BLANCA', name: 'EOSINOFILOS', result: '0.7', unit: 'x mil/uL', refRange: '0 - 0.5', status: 'alerta', isHighlighted: true, description: 'Recuento absoluto de eosinófilos. Leve elevación a evaluar con síntomas alérgicos.' },
      { group: 'SERIE BLANCA', name: 'BASOFILOS', result: '0.1', unit: 'x mil/uL', refRange: '0 - 0.1', status: 'normal', description: 'Recuento absoluto de basófilos en sangre.' },
      // SERIE PLAQUETAR
      { group: 'SERIE PLAQUETAR', name: 'PLAQUETAS', result: '263', unit: 'x mil/mm3', refRange: '130 - 400', status: 'optimo', description: 'Fragmentos celulares esenciales para la coagulación y la prevención de hemorragias.' },
      { group: 'SERIE PLAQUETAR', name: 'VOLUMEN PLAQUETARIO MEDIO', result: '12.1', unit: 'fL', refRange: '9 - 13', status: 'normal', description: 'Tamaño promedio de las plaquetas; refleja la tasa de producción plaquetaria medular.' }
    ],
    clinicalNote: 'Serie roja y serie blanca dentro de los límites biológicos de referencia. Leve eosinofilia a correlacionar clínicamente.'
  },
  {
    id: 2,
    name: 'DESCARTE DE VIH 1/2 + ANTÍGENO P24',
    category: 'Inmunoserología & Infectología',
    deliveryTime: 'Mismo Día (Tarde)',
    fastingHours: '4 Horas',
    sampleType: 'Suero Sanguíneo (Tubo Rojo/Amarillo con Gel)',
    shortDesc: 'Detección simultánea del antígeno p24 y anticuerpos contra VIH-1 y VIH-2. Reduce drásticamente el periodo de ventana clínica.',
    methodology: 'Quimioluminiscencia Microparticular (CMIA de 4ta Generación)',
    biomarkers: [
      { group: 'SEROLOGÍA VIRAL', name: 'ANTICUERPOS ANTI-VIH 1/2', result: '0.12', unit: 'Índice S/CO', refRange: '< 0.90 Negativo', status: 'optimo', description: 'Anticuerpos producidos por el sistema inmunitario contra el virus VIH de tipo 1 o tipo 2.' },
      { group: 'SEROLOGÍA VIRAL', name: 'ANTÍGENO CAPSULAR P24', result: '0.08', unit: 'Índice S/CO', refRange: '< 0.90 Negativo', status: 'optimo', description: 'Proteína del núcleo del virus que aparece tempranamente antes de que se creen los anticuerpos.' },
      { group: 'CONCLUSIÓN SEROLÓGICA', name: 'RESULTADO DIAGNÓSTICO', result: 'NO REACTIVO', unit: '-', refRange: 'No Reactivo', status: 'optimo', description: 'Conclusión final de la prueba por método de quimioluminiscencia de 4ta generación.' }
    ],
    clinicalNote: 'RESULTADO: NO REACTIVO. No se detectan anticuerpos ni antígeno p24 para el virus de inmunodeficiencia humana en la muestra remitida.'
  },
  {
    id: 3,
    name: 'GLUCOSA Y HEMOGLOBINA GLICOSILADA (HBA1C)',
    category: 'Bioquímica Metabólica',
    deliveryTime: '4 Horas',
    fastingHours: '8 a 10 Horas estricto',
    sampleType: 'Plasma Fluoruro / Sangre EDTA',
    shortDesc: 'Monitoreo de glucosa inmediata y hemoglobina glicosilada certificada NGSP para el promedio glucémico de los últimos 90 días.',
    methodology: 'Hexoquinasa Enzimática & HPLC Cromatografía Líquida',
    biomarkers: [
      { group: 'METABOLISMO GLUCÍDICO', name: 'GLUCOSA BASAL EN AYUNAS', result: '88.0', unit: 'mg/dL', refRange: '70.0 - 99.0', status: 'optimo', description: 'Nivel actual de azúcar circulante en la sangre tras un periodo de ayuno nocturno.' },
      { group: 'METABOLISMO GLUCÍDICO', name: 'HEMOGLOBINA GLICOSILADA (HbA1c)', result: '5.2', unit: '%', refRange: '< 5.7 Normal', status: 'optimo', description: 'Porcentaje de hemoglobina unida a glucosa; refleja el promedio del azúcar en sangre de los últimos 3 meses.' },
      { group: 'ESTIMACIÓN CLÍNICA', name: 'GLUCOSA PROMEDIO ESTIMADA (eAG)', result: '103.0', unit: 'mg/dL', refRange: '< 117.0', status: 'normal', description: 'Cálculo matemático correlativo del promedio diario de glucosa basado en la HbA1c.' }
    ],
    clinicalNote: 'Control glucémico excelente. Valores compatibles con metabolismo de carbohidratos normal y sin criterios de prediabetes.'
  },
  {
    id: 4,
    name: 'PERFIL LIPÍDICO COMPLETO',
    category: 'Salud Cardiovascular',
    deliveryTime: 'Mismo Día',
    fastingHours: '10 a 12 Horas estricto',
    sampleType: 'Suero Sanguíneo sin Hemólisis',
    shortDesc: 'Fraccionamiento de lipoproteínas circulantes para evaluar el riesgo aterogénico y salud coronaria.',
    methodology: 'Enzimático Colorimétrico Automatizado Trinder',
    biomarkers: [
      { group: 'FRACCIONES LIPÍDICAS', name: 'COLESTEROL TOTAL', result: '178.0', unit: 'mg/dL', refRange: '< 200.0 Deseable', status: 'optimo', description: 'Cantidad total de colesterol presente en la sangre (suma de todas las lipoproteínas).' },
      { group: 'FRACCIONES LIPÍDICAS', name: 'COLESTEROL HDL (CARDIOPROTECTOR)', result: '55.0', unit: 'mg/dL', refRange: '> 40.0 Óptimo', status: 'optimo', description: 'Conocido como colesterol "bueno"; transporta la grasa desde las arterias de regreso al hígado.' },
      { group: 'FRACCIONES LIPÍDICAS', name: 'COLESTEROL LDL (CALCULADO)', result: '98.0', unit: 'mg/dL', refRange: '< 100.0 Óptimo', status: 'optimo', description: 'Conocido como colesterol "malo"; su exceso se deposita en las paredes de las arterias formando placas.' },
      { group: 'FRACCIONES LIPÍDICAS', name: 'TRIGLICÉRIDOS', result: '125.0', unit: 'mg/dL', refRange: '< 150.0 Normal', status: 'normal', description: 'Grasas de reserva energética; niveles elevados incrementan el riesgo cardiovascular y pancreatitis.' },
      { group: 'ÍNDICES DE RIESGO', name: 'ÍNDICE CASTELLI (COL T / HDL)', result: '3.2', unit: 'Ratio', refRange: '< 4.5 Bajo Riesgo', status: 'optimo', description: 'Relación matemática entre Colesterol Total y HDL para estimar el riesgo de aterosclerosis.' }
    ],
    clinicalNote: 'Perfil de lípidos dentro del rango cardiosaludable. Riesgo de enfermedad vascular coronaria estimado en nivel bajo.'
  },
  {
    id: 5,
    name: 'UROANÁLISIS COMPLETO (EXAMEN DE ORINA)',
    category: 'Nefrología & Vías Urinarias',
    deliveryTime: '2 Horas',
    fastingHours: 'No indispensable',
    sampleType: 'Primera orina de la mañana (Chorro medio estéril)',
    shortDesc: 'Evaluación física, bioquímica con reflectancia fotométrica y microscopía óptica del sedimento urinario centrifugado.',
    methodology: 'Tira Reactiva Multizona & Microscopía Automatizada de Elementos Formes',
    biomarkers: [
      { group: 'EXAMEN FÍSICO-QUÍMICO', name: 'ASPECTO Y COLOR', result: 'LÍMPIDO / AMARILLO', unit: '-', refRange: 'Límpido / Pajizo', status: 'normal', description: 'Inspección visual macroscópica; evalúa hidratación, concentración y presencia de turbidez.' },
      { group: 'EXAMEN FÍSICO-QUÍMICO', name: 'DENSIDAD ESPECÍFICA', result: '1.020', unit: '-', refRange: '1.010 - 1.030', status: 'normal', description: 'Capacidad de concentración y dilución de los riñones.' },
      { group: 'EXAMEN FÍSICO-QUÍMICO', name: 'PROTEÍNAS / ALBÚMINA', result: 'NEGATIVO', unit: 'mg/dL', refRange: 'Negativo', status: 'optimo', description: 'Descarte de filtración de proteínas glomerulares a través de los riñones.' },
      { group: 'SEDIMENTO MICROSCÓPICO', name: 'LEUCOCITOS (GLÓBULOS BLANCOS)', result: '1 - 2', unit: 'x Campo', refRange: '0 - 5 x Campo', status: 'optimo', description: 'Células inmunes en orina; valores altos sugieren infección de vías urinarias (cistitis/pielonefritis).' },
      { group: 'SEDIMENTO MICROSCÓPICO', name: 'BACTERIAS', result: 'ESCASAS', unit: '-', refRange: 'Escasas o Ausentes', status: 'normal', description: 'Presencia o ausencia de microorganismos patógenos en el sedimento urinario.' }
    ],
    clinicalNote: 'Examen de orina negativo para proceso infeccioso activo. Ausencia de cilindros patológicos, hematíes isomórficos ni proteinuria.'
  },
  {
    id: 6,
    name: 'DESCARTE DE SÍFILIS (VDRL / RPR)',
    category: 'Inmunoserología ETS',
    deliveryTime: 'Mismo Día',
    fastingHours: '4 Horas',
    sampleType: 'Suero Sanguíneo',
    shortDesc: 'Prueba no treponémica floculante para detección de reaginas asociadas a Treponema pallidum.',
    methodology: 'Microfloculación en Lámina RPR / VDRL',
    biomarkers: [
      { group: 'INMUNOLOGÍA TREPONÉMICA', name: 'REAGINA SÉRICA (RPR)', result: 'NO REACTIVO', unit: '-', refRange: 'No Reactivo', status: 'optimo', description: 'Prueba no treponémica rápida de floculación para descarte activo de sífilis.' },
      { group: 'INMUNOLOGÍA TREPONÉMICA', name: 'TÍTULO TREPONÉMICO', result: '0 Dils', unit: 'Dilución', refRange: 'Sin titulación', status: 'optimo', description: 'Grado de dilución reactiva para cuantificar la carga de anticuerpos reamínicos.' }
    ],
    clinicalNote: 'No se observó aglutinación de antígeno cardiolipina. Resultado compatible con ausencia de infección activa por lúes.'
  }
]

const selectedService = ref<ServiceItem>(servicesList[0])
const activeBiomarkerDetail = ref<Biomarker | null>(null)

// 8 Slides de Carrusel
const slides: Slide[] = [
  {
    id: 1,
    category: 'Hematología Clínica',
    title: 'Análisis de Sangre Completo',
    subtitle: 'Hemograma Automatizado',
    description: 'Extracción venosa suave y conteo globular automatizado. Mide glóbulos rojos, blancos, hemoglobina, plaquetas y descartes de procesos infecciosos o anemias.',
    image: 'https://images.unsplash.com/photo-1615461066841-6116e61058f4?auto=format&fit=crop&w=1920&q=80',
    badgeText: 'Entrega en 3 a 4 horas',
    statNumber: '3 - 4 hrs',
    statLabel: 'Tiempo de Entrega',
    secondaryStatNumber: '4 hrs',
    secondaryStatLabel: 'Ayuno ligero requerido',
    accentColor: '#00B4FF'
  },
  {
    id: 2,
    category: 'Infectología & Serología',
    title: 'Prueba de Descarte de VIH',
    subtitle: 'ELISA 4ta Gen / Rápida',
    description: 'Detección cuantitativa de anticuerpos y antígeno p24 para máxima precocidad y sensibilidad. Procesamiento confidencial y seguro bajo código de privacidad.',
    image: 'https://images.unsplash.com/photo-1579154204601-01588f351e67?auto=format&fit=crop&w=1920&q=80',
    badgeText: '100% Confidencial',
    statNumber: '99.9%',
    statLabel: 'Sensibilidad diagnóstica',
    secondaryStatNumber: 'Mismo día',
    secondaryStatLabel: 'Informe privado y seguro',
    accentColor: '#00B4FF'
  },
  {
    id: 3,
    category: 'Uroanálisis & Función Renal',
    title: 'Análisis de Orina Completo',
    subtitle: 'Sedimento y Tira Reactiva',
    description: 'Examen físico, químico y sedimentario microscópico en frasco estéril. Evalúa cristales, leucocituria, hematuria, proteínas y descarte de infecciones renales o de vías urinarias.',
    image: 'https://images.unsplash.com/photo-1582719478250-c89cae4dc85b?auto=format&fit=crop&w=1920&q=80',
    badgeText: 'Resultados rápidos',
    statNumber: '2 horas',
    statLabel: 'Tiempo promedio',
    secondaryStatNumber: '1ra orina',
    secondaryStatLabel: 'Chorro medio recomendado',
    accentColor: '#00B4FF'
  },
  {
    id: 4,
    category: 'Bioquímica Metabólica',
    title: 'Análisis de Glucosa y Diabetes',
    subtitle: 'Glucemia Basal & HbA1c',
    description: 'Monitoreo de glucosa en plasma venoso y hemoglobina glicosilada (HbA1c) certificada NGSP. Permite evaluar tu promedio real de azúcar de los últimos 3 meses.',
    image: 'https://images.unsplash.com/photo-1505751172876-fa1923c5c528?auto=format&fit=crop&w=1920&q=80',
    badgeText: 'Control glucémico',
    statNumber: '8 hrs',
    statLabel: 'Ayuno previo',
    secondaryStatNumber: 'NGSP',
    secondaryStatLabel: 'Metodología certificada',
    accentColor: '#00B4FF'
  },
  {
    id: 5,
    category: 'Salud Cardiovascular',
    title: 'Colesterol y Perfil Lipídico',
    subtitle: 'Riesgo Coronario y Arterial',
    description: 'Determinación cuantitativa de Colesterol Total, HDL (cardioprotector), LDL (colesterol malo), VLDL y Triglicéridos. Esencial para prevenir infartos e hipertensión.',
    image: 'https://images.unsplash.com/photo-1628348068343-c6a848d2b6dd?auto=format&fit=crop&w=1920&q=80',
    badgeText: 'Prevención infartos',
    statNumber: '10-12 h',
    statLabel: 'Ayuno estricto',
    secondaryStatNumber: '5 Parám.',
    secondaryStatLabel: 'Perfil lípido completo',
    accentColor: '#00B4FF'
  },
  {
    id: 6,
    category: 'Inmunoserología ETS',
    title: 'Descarte de Sífilis (VDRL / RPR)',
    subtitle: 'Pruebas Treponémicas',
    description: 'Análisis serológico para detección de anticuerpos contra Treponema pallidum. Utilizado para despistajes preventivos de salud sexual, chequeos prenupciales y prenatales.',
    image: 'https://images.unsplash.com/photo-1516549655169-df83a0774514?auto=format&fit=crop&w=1920&q=80',
    badgeText: 'Atención discreta',
    statNumber: 'Mismo día',
    statLabel: 'Entrega de informe',
    secondaryStatNumber: 'RPR/TPHA',
    secondaryStatLabel: 'Prueba dual de certeza',
    accentColor: '#00B4FF'
  },
  {
    id: 7,
    category: 'Microbiología & Biología Molecular',
    title: 'Gonorrea y Clamidia (PCR)',
    subtitle: 'Detección Molecular Rápida',
    description: 'Prueba molecular de alta sensibilidad para Neisseria gonorrhoeae y Chlamydia trachomatis en muestra de orina o hisopado. Diagnóstico certero y oportuno.',
    image: 'https://images.unsplash.com/photo-1579684385127-1ef15d508118?auto=format&fit=crop&w=1920&q=80',
    badgeText: 'Biología molecular',
    statNumber: 'PCR',
    statLabel: 'Máxima especificidad',
    secondaryStatNumber: '24 hrs',
    secondaryStatLabel: 'Procesamiento especializado',
    accentColor: '#00B4FF'
  },
  {
    id: 8,
    category: 'Virología Especializada',
    title: 'Descarte de Herpes Virus (HSV 1 y 2)',
    subtitle: 'Anticuerpos IgG / IgM',
    description: 'Diferenciación serológica de anticuerpos específicos para Herpes Simplex 1 (oral) y Tipo 2 (genital). Informes claros para definir fase aguda o latente.',
    image: 'https://images.unsplash.com/photo-1532187863486-abf9dbad1b69?auto=format&fit=crop&w=1920&q=80',
    badgeText: 'Diferenciación Tipo 1 y 2',
    statNumber: 'IgG / IgM',
    statLabel: 'Titulaciones cuantitativas',
    secondaryStatNumber: '24 hrs',
    secondaryStatLabel: 'Reporte clínico detallado',
    accentColor: '#00B4FF'
  }
]

const activeIndex = ref<number>(0)
const isModalOpen = ref<boolean>(false)
const thumbnailsContainer = ref<HTMLElement | null>(null)

let autoSlideTimer: number | null = null
const SLIDE_DURATION_MS = 6000

// DESPLAZAMIENTO SUAVE PURAMENTE HORIZONTAL (NUNCA TOCA EL SCROLL VERTICAL DE LA PÁGINA)
const scrollThumbnailsHorizontally = () => {
  if (thumbnailsContainer.value) {
    const activeEl = thumbnailsContainer.value.children[activeIndex.value] as HTMLElement
    if (activeEl) {
      const containerLeft = thumbnailsContainer.value.getBoundingClientRect().left
      const elementLeft = activeEl.getBoundingClientRect().left
      const currentScroll = thumbnailsContainer.value.scrollLeft
      const targetScroll = currentScroll + (elementLeft - containerLeft) - (thumbnailsContainer.value.clientWidth / 2) + (activeEl.clientWidth / 2)
      
      thumbnailsContainer.value.scrollTo({
        left: targetScroll,
        behavior: 'smooth'
      })
    }
  }
}

const nextSlide = () => {
  activeIndex.value = (activeIndex.value + 1) % slides.length
  scrollThumbnailsHorizontally()
  resetTimer()
}

const prevSlide = () => {
  activeIndex.value = (activeIndex.value - 1 + slides.length) % slides.length
  scrollThumbnailsHorizontally()
  resetTimer()
}

const goToSlide = (index: number) => {
  activeIndex.value = index
  scrollThumbnailsHorizontally()
  resetTimer()
}

const startAutoPlay = () => {
  if (autoSlideTimer) clearInterval(autoSlideTimer)
  autoSlideTimer = window.setInterval(() => {
    // SÓLO AVANZA AUTOMÁTICAMENTE SI EL USUARIO ESTÁ ARRIBA EN EL HERO (scrollY < 300)
    // SI EL USUARIO HIZO SCROLL HACIA ABAJO PARA LEER OTRAS SECCIONES, NO LO INTERRUMPE NI LO MUEVE
    if (!isModalOpen.value && window.scrollY < 300) {
      activeIndex.value = (activeIndex.value + 1) % slides.length
      scrollThumbnailsHorizontally()
    }
  }, SLIDE_DURATION_MS)
}

const resetTimer = () => {
  startAutoPlay()
}

const isScrolled = ref(false)
const scrollProgress = ref(0) // Porcentaje de scroll de 0 a 100

const handleScroll = () => {
  isScrolled.value = window.scrollY > 30
  
  // Calcular porcentaje de scroll para pintar el borde del botón
  const winScroll = document.documentElement.scrollTop || document.body.scrollTop
  const height = document.documentElement.scrollHeight - document.documentElement.clientHeight
  if (height > 0) {
    scrollProgress.value = Math.min(100, Math.max(0, Math.round((winScroll / height) * 100)))
  } else {
    scrollProgress.value = 0
  }
}

const scrollToTop = () => {
  window.scrollTo({
    top: 0,
    behavior: 'smooth'
  })
}

onMounted(() => {
  startAutoPlay()
  window.addEventListener('scroll', handleScroll, { passive: true })
  nextTick(() => {
    initMap()
  })
})

onUnmounted(() => {
  window.removeEventListener('scroll', handleScroll)
  if (autoSlideTimer) clearInterval(autoSlideTimer)
  if (leafletMap) {
    leafletMap.remove()
    leafletMap = null
  }
})

const openModal = () => {
  isModalOpen.value = true
}

const closeModal = () => {
  isModalOpen.value = false
}

const handleResultsSubmit = (e: Event) => {
  e.preventDefault()
  alert('Descargando informe clínico oficial cifrado...')
  closeModal()
}
</script>

<template>
  <div class="relative w-full bg-slate-900 text-white selection:bg-[#00B4FF] selection:text-[#002B94]">
    
    <!-- ========================================================================= -->
    <!-- SECCIÓN 1: HERO CON CARRUSEL -->
    <!-- ========================================================================= -->
    <section class="relative w-full min-h-screen flex flex-col justify-between overflow-hidden">
      <!-- FONDO NÍTIDO -->
      <div class="absolute inset-0 z-0">
        <div 
          v-for="(slide, idx) in slides" 
          :key="slide.id"
          class="absolute inset-0 transition-opacity duration-1000 ease-in-out"
          :class="activeIndex === idx ? 'opacity-100 scale-100' : 'opacity-0 scale-105 pointer-events-none'"
          style="transition-property: opacity, transform;"
        >
          <img :src="slide.image" :alt="slide.title" class="w-full h-full object-cover object-center" />
          <div class="absolute inset-0" style="background: linear-gradient(90deg, rgba(0, 15, 50, 0.78) 0%, rgba(0, 25, 80, 0.4) 45%, transparent 80%);"></div>
          <div class="absolute inset-0" style="background: linear-gradient(180deg, rgba(0, 10, 35, 0.5) 0%, transparent 20%, transparent 60%, rgba(0, 10, 35, 0.85) 100%);"></div>
        </div>
      </div>

      <!-- ========================================================================= -->
      <!-- HEADER CON IDENTIDAD VISUAL BIOMEDILAB OFICIAL (ISOTIPO IZQ + TEXTO DER) -->
      <!-- ========================================================================= -->
      <header 
        class="fixed top-0 left-0 right-0 z-50 transition-all duration-300"
        :class="[
          isScrolled 
            ? 'bg-[#00123b]/95 backdrop-blur-2xl py-2.5 sm:py-3 border-b border-[#00B4FF]/20 shadow-[0_12px_40px_rgba(0,10,35,0.85)]' 
            : 'bg-gradient-to-b from-[#000d2b]/95 via-[#001442]/80 to-transparent py-3.5 sm:py-5'
        ]"
      >
        <div class="max-w-[98%] 2xl:max-w-[1680px] mx-auto px-4 sm:px-8">
          <div class="flex items-center justify-between gap-6">
            
            <!-- NUEVO LOGOTIPO OFICIAL: ICONO A LA IZQUIERDA Y TEXTO A LA DERECHA EN UNA SOLA IMAGEN NÍTIDA -->
            <a href="#" class="flex items-center group shrink-0 select-none py-1">
              <img 
                src="/logo-horizontal-biomedilab.png" 
                alt="Biomedilab Laboratorio Clínico" 
                class="h-12 sm:h-14 md:h-16 w-auto object-contain transition-all duration-300 group-hover:scale-[1.03] filter drop-shadow-[0_4px_16px_rgba(0,180,255,0.45)]" 
              />
            </a>

            <!-- NAVEGACIÓN EN PÍLDORA ELEGANTE -->
            <div class="hidden lg:flex items-center">
              <nav class="flex items-center gap-1 p-1.5 rounded-full bg-white/[0.06] backdrop-blur-md border border-white/10 shadow-inner">
                <a 
                  href="#" 
                  class="px-5 py-2 rounded-full text-xs font-black uppercase tracking-wider text-[#001D69] bg-[#00B4FF] shadow-[0_2px_12px_rgba(0,180,255,0.4)] transition-all"
                >
                  Inicio
                </a>
                <a 
                  href="#especialistas" 
                  class="px-4 py-2 rounded-full text-xs font-bold uppercase tracking-wider text-white/80 hover:text-white hover:bg-white/10 transition-all"
                >
                  Especialistas
                </a>
                <a 
                  href="#servicios-hoja" 
                  class="px-4 py-2 rounded-full text-xs font-bold uppercase tracking-wider text-white/80 hover:text-white hover:bg-white/10 transition-all"
                >
                  Servicios
                </a>
                <a 
                  href="#sedes-mapa" 
                  class="px-4 py-2 rounded-full text-xs font-bold uppercase tracking-wider text-white/80 hover:text-[#00B4FF] hover:bg-white/10 transition-all flex items-center gap-1.5"
                >
                  <MapPin class="w-3.5 h-3.5 text-[#00B4FF]" />
                  <span>Nuestras Sedes</span>
                </a>
              </nav>
            </div>

            <!-- ACCIONES LATERALES: TELÉFONO + DESCARGAR RESULTADOS + SEPARAR CITA -->
            <div class="flex items-center gap-3 sm:gap-4 shrink-0">
              <a 
                href="tel:015550100" 
                class="hidden xl:flex items-center gap-2 px-3.5 py-2 rounded-full text-xs font-bold text-white/85 hover:text-white hover:bg-white/5 border border-transparent hover:border-white/10 transition-all"
              >
                <div class="w-7 h-7 rounded-full bg-[#00B4FF]/15 flex items-center justify-center text-[#00B4FF]">
                  <Phone class="w-3.5 h-3.5" />
                </div>
                <span>(01) 555-0100</span>
              </a>

              <button 
                @click="openModal" 
                class="hidden sm:flex items-center gap-2 px-4 py-2.5 rounded-full text-xs font-extrabold uppercase tracking-wide text-white bg-white/10 hover:bg-white/20 border border-white/20 hover:border-[#00B4FF]/50 transition-all cursor-pointer shadow-sm active:scale-95"
              >
                <FileCheck2 class="w-4 h-4 text-[#00B4FF]" />
                <span>Resultados</span>
              </button>

              <a 
                :href="`https://wa.me/51999999999?text=Hola%20Biomedilab,%20deseo%20separar%20cita%20para%20${encodeURIComponent(slides[activeIndex].title)}`" 
                target="_blank" 
                class="px-5 sm:px-6 py-2.5 rounded-full text-xs sm:text-sm font-black text-[#001D69] bg-gradient-to-r from-[#00B4FF] to-[#38c8ff] hover:from-white hover:to-white hover:text-[#001D69] transition-all shadow-[0_4px_22px_rgba(0,180,255,0.5)] flex items-center gap-2 active:scale-95 cursor-pointer whitespace-nowrap"
              >
                <Calendar class="w-4 h-4 text-[#001D69]" />
                <span>Separar Cita</span>
              </a>
            </div>

          </div>
        </div>
      </header>

      <!-- CONTENIDO CENTRAL HERO -->
      <main class="relative z-20 max-w-7xl mx-auto w-full px-6 sm:px-10 lg:px-14 flex-1 flex flex-col justify-center py-6 lg:py-10">
        <div class="max-w-2xl">
          <div class="inline-flex items-center gap-2 px-3.5 py-1.5 rounded-full bg-slate-900/60 border border-white/20 backdrop-blur-md mb-4">
            <span class="w-2 h-2 rounded-full bg-[#00B4FF] animate-pulse"></span>
            <span class="text-xs font-bold text-[#00B4FF] uppercase tracking-wider">{{ slides[activeIndex].category }}</span>
            <span class="text-white/40">|</span>
            <span class="text-xs text-white/90">{{ slides[activeIndex].badgeText }}</span>
          </div>

          <h1 class="font-['Outfit'] font-black text-4xl sm:text-5xl lg:text-6xl text-white leading-[1.08] tracking-tight mb-4 transition-all duration-500 drop-shadow-lg">
            {{ slides[activeIndex].title }}
          </h1>

          <p class="text-base sm:text-lg text-white/90 leading-relaxed mb-6 max-w-xl transition-all duration-500 drop-shadow">
            {{ slides[activeIndex].description }}
          </p>

          <div class="flex items-center gap-4">
            <a :href="`https://wa.me/51999999999?text=Hola%20Biomedilab,%20deseo%20separar%20cita%20para%20${encodeURIComponent(slides[activeIndex].title)}`" target="_blank" class="px-8 py-3.5 rounded-full bg-[#004DE6] hover:bg-[#0038B8] text-white font-black text-sm tracking-wide transition-all shadow-[0_8px_25px_rgba(0,77,230,0.5)] flex items-center gap-2">
              <span>Separar Cita</span>
              <ArrowRight class="w-4 h-4" />
            </a>
          </div>
        </div>
      </main>

      <!-- ZONA INFERIOR HERO: BOTONES Y CARRUSEL -->
      <div class="relative z-20 w-full mb-8 sm:mb-12 px-6 sm:px-10 lg:px-14">
        <div class="max-w-7xl mx-auto flex flex-col lg:flex-row lg:items-end justify-between gap-8">
          <div class="flex flex-col gap-4 shrink-0">
            <div class="flex flex-wrap items-center gap-3.5">
              <a :href="`https://wa.me/51999999999?text=Hola%20Biomedilab,%20deseo%20separar%20cita%20para:%20${encodeURIComponent(slides[activeIndex].title)}`" target="_blank" class="px-7 py-3.5 rounded-2xl bg-[#00B4FF] hover:bg-white text-[#002B94] font-black text-sm transition-all duration-200 shadow-[0_0_25px_rgba(0,180,255,0.5)] flex items-center gap-2.5 active:scale-95">
                <MessageCircle class="w-5 h-5 text-[#002B94]" />
                <span>Separar Cita</span>
              </a>
              <button @click="openModal" class="px-7 py-3.5 rounded-2xl bg-white/10 hover:bg-white/20 border border-white/30 backdrop-blur-md text-white font-extrabold text-sm transition-all flex items-center gap-2.5 active:scale-95 shadow-sm">
                <Info class="w-5 h-5 text-[#00B4FF]" />
                <span>Más Información</span>
              </button>
            </div>

            <div class="flex items-center gap-6 text-xs text-white/85">
              <div>
                <p class="font-bold text-white text-base drop-shadow">{{ slides[activeIndex].statNumber }}</p>
                <p class="text-[11px] text-white/70">{{ slides[activeIndex].statLabel }}</p>
              </div>
              <div class="h-6 w-px bg-white/20"></div>
              <div>
                <p class="font-bold text-[#00B4FF] text-base drop-shadow">{{ slides[activeIndex].secondaryStatNumber }}</p>
                <p class="text-[11px] text-white/70">{{ slides[activeIndex].secondaryStatLabel }}</p>
              </div>
            </div>
          </div>

          <!-- Carrusel de Miniaturas -->
          <div class="flex flex-col sm:items-end gap-3 flex-1 min-w-0 max-w-full lg:max-w-2xl">
            <div class="w-full flex items-center justify-between sm:justify-end gap-6 text-sm text-white font-bold">
              <div class="flex items-center gap-2 font-mono drop-shadow">
                <span class="text-[#00B4FF] text-base">0{{ activeIndex + 1 }}</span>
                <span class="text-white/40">—</span>
                <span class="text-white/80">0{{ slides.length }}</span>
              </div>
              <div class="flex items-center gap-2">
                <button @click="prevSlide" class="w-10 h-10 rounded-full bg-white/20 hover:bg-[#004DE6] hover:text-white border border-white/30 flex items-center justify-center transition-all cursor-pointer shadow-md active:scale-95 backdrop-blur-sm">
                  <ArrowLeft class="w-5 h-5" />
                </button>
                <button @click="nextSlide" class="w-10 h-10 rounded-full bg-white/20 hover:bg-[#004DE6] hover:text-white border border-white/30 flex items-center justify-center transition-all cursor-pointer shadow-md active:scale-95 backdrop-blur-sm">
                  <ArrowRight class="w-5 h-5" />
                </button>
              </div>
            </div>

            <div ref="thumbnailsContainer" class="w-full flex items-center gap-4 overflow-x-auto py-4 px-3 max-w-full no-scrollbar scroll-smooth">
              <button 
                v-for="(slide, index) in slides" 
                :key="slide.id"
                @click="goToSlide(index)"
                class="relative rounded-2xl overflow-hidden shrink-0 transition-all duration-300 group text-left cursor-pointer"
                :class="activeIndex === index ? 'w-36 sm:w-44 h-24 sm:h-28 ring-4 ring-[#00B4FF] shadow-[0_10px_25px_rgba(0,180,255,0.7)] scale-105' : 'w-28 sm:w-36 h-20 sm:h-24 opacity-65 hover:opacity-100 hover:scale-100 border border-white/25'"
              >
                <img :src="slide.image" :alt="slide.title" class="w-full h-full object-cover transition-transform duration-500 group-hover:scale-110" />
                <div class="absolute inset-0 bg-gradient-to-t from-slate-950/90 via-slate-950/20 to-transparent"></div>
                <span class="absolute bottom-2 left-2.5 right-2.5 text-[11px] sm:text-xs font-bold text-white truncate block drop-shadow leading-tight">{{ slide.subtitle }}</span>
              </button>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- ========================================================================= -->
    <!-- SECCIÓN 2: NUESTROS ESPECIALISTAS Y PATÓLOGOS CLÍNICOS -->
    <!-- ========================================================================= -->
    <section id="especialistas" class="py-24 bg-white text-slate-800">
      <div class="max-w-7xl mx-auto px-6 sm:px-10 lg:px-14">
        <div class="flex flex-col sm:flex-row items-start sm:items-center justify-between gap-4 mb-16">
          <div>
            <div class="inline-flex items-center gap-2 px-3 py-1 rounded-full bg-[#EBF7FF] text-[#004DE6] text-xs font-bold uppercase tracking-wider mb-2">
              <span>Equipo Médico Certificado</span>
            </div>
            <h2 class="font-['Outfit'] font-black text-3xl sm:text-4xl lg:text-5xl text-[#001D69] tracking-tight">
              Solo Contamos con <br class="hidden sm:inline" />Especialistas Acreditados
            </h2>
          </div>
          <a href="https://wa.me/51999999999?text=Hola%20Biomedilab,%20deseo%20conocer%20la%20lista%20completa%20de%20especialistas" target="_blank" class="inline-flex items-center gap-2.5 px-6 py-3 rounded-2xl bg-[#00B4FF] hover:bg-[#0092D1] text-white font-extrabold text-xs tracking-wide transition-all shadow-[0_4px_15px_rgba(0,180,255,0.35)] active:scale-95">
            <span>Ver Todos los Especialistas</span>
            <span class="w-6 h-6 rounded-full bg-white/20 flex items-center justify-center">
              <ArrowRight class="w-3.5 h-3.5 text-white" />
            </span>
          </a>
        </div>

        <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-6 items-start pb-8">
          <div 
            v-for="doctor in specialists" 
            :key="doctor.id"
            @mouseenter="hoveredSpecialist = doctor.id"
            class="relative rounded-[28px] overflow-hidden transition-all duration-300 flex flex-col justify-between cursor-pointer"
            :class="hoveredSpecialist === doctor.id ? 'bg-[#EBF7FF] border-[4px] border-[#001D69] shadow-[0_20px_45px_rgba(0,29,105,0.22)] -translate-y-2' : 'bg-[#F0F6FA] border-[1.5px] border-slate-200/90 hover:border-slate-300'"
          >
            <div class="relative w-full h-80 overflow-hidden bg-[#F8FAFC] flex items-center justify-center">
              <img :src="doctor.image" :alt="doctor.name" class="w-full h-full object-cover object-top transition-transform duration-500" :class="hoveredSpecialist === doctor.id ? 'scale-105' : 'scale-100'" />
              
              <!-- Etiqueta de Colegiatura Médica -->
              <div class="absolute top-3 left-3 bg-[#001D69]/85 backdrop-blur-md px-2.5 py-1 rounded-lg text-[10px] font-bold text-white border border-white/20">
                {{ doctor.colegiatura }}
              </div>

              <div v-if="hoveredSpecialist === doctor.id" class="absolute bottom-3 left-4 right-4 z-10 animate-in fade-in duration-200">
                <a :href="`https://wa.me/51999999999?text=Hola,%20deseo%20agendar%20atenci%C3%B3n%20con%20${encodeURIComponent(doctor.name)}`" target="_blank" class="w-full py-3 px-4 rounded-xl bg-[#00B4FF] hover:bg-[#0095D4] text-white font-extrabold text-xs flex items-center justify-center gap-2 shadow-md transition-all active:scale-95">
                  <Calendar class="w-4 h-4" />
                  <span>Agendar Consulta Ahora</span>
                </a>
              </div>
            </div>

            <div class="p-5 flex items-center justify-between transition-colors duration-300" :class="hoveredSpecialist === doctor.id ? 'bg-[#001D69] text-white' : 'bg-[#E7F0F7] text-slate-800'">
              <div>
                <h3 class="font-['Outfit'] font-black text-base leading-tight">{{ doctor.name }}</h3>
                <p class="text-xs font-semibold mt-1" :class="hoveredSpecialist === doctor.id ? 'text-[#00B4FF]' : 'text-[#004DE6]'">{{ doctor.role }}</p>
                <p v-if="hoveredSpecialist === doctor.id" class="text-[10px] text-white/70 mt-1 line-clamp-1">{{ doctor.specialtyDetail }}</p>
              </div>
              <div class="w-9 h-9 rounded-xl flex items-center justify-center transition-all shrink-0 ml-2" :class="hoveredSpecialist === doctor.id ? 'bg-white text-[#001D69]' : 'bg-[#001D69] text-white'">
                <ArrowRight class="w-4 h-4" />
              </div>
            </div>

            <div v-if="hoveredSpecialist === doctor.id" class="bg-[#F0F6FA] py-3 px-6 flex items-center justify-center gap-5 text-[#001D69] border-t border-slate-200 animate-in fade-in duration-200">
              <a href="#" class="hover:text-[#00B4FF] transition-colors"><Linkedin class="w-3.5 h-3.5" /></a>
              <a href="#" class="hover:text-[#00B4FF] transition-colors"><Instagram class="w-3.5 h-3.5" /></a>
              <a href="#" class="hover:text-[#00B4FF] transition-colors"><Facebook class="w-3.5 h-3.5" /></a>
              <a href="#" class="hover:text-[#00B4FF] transition-colors font-bold text-xs">𝕏</a>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- ========================================================================= -->
    <!-- SECCIÓN 3: SERVICIOS CLÍNICOS, HOJA DE RESULTADOS Y PREPARACIÓN (PANORÁMICO MAX-W-[1600PX]) -->
    <!-- ========================================================================= -->
    <section id="servicios-hoja" class="py-20 bg-[#F4F8FB] text-slate-800 border-t border-slate-200">
      <div class="max-w-[98%] 2xl:max-w-[1600px] mx-auto px-2 sm:px-4 lg:px-6">
        
        <div class="flex flex-col md:flex-row md:items-end justify-between gap-4 mb-8">
          <div>
            <div class="inline-flex items-center gap-2 px-3 py-1 rounded-full bg-[#001D69] text-[#00B4FF] text-xs font-black uppercase tracking-wider mb-2">
              <FileSpreadsheet class="w-3.5 h-3.5" />
              <span>Catálogo Interactivo y Ficha Técnica</span>
            </div>
            <h2 class="font-['Outfit'] font-black text-2xl sm:text-4xl text-[#001D69] tracking-tight">
              Servicios Clínicos, Hoja de Resultados y Preparación
            </h2>
            <p class="text-xs sm:text-sm text-slate-600 mt-1 max-w-2xl">
              Selecciona cualquier examen en el catálogo para visualizar la hoja de informe oficial, valores de referencia y requisitos clínicos de toma de muestra.
            </p>
          </div>

          <div class="flex items-center gap-2 text-xs font-bold text-slate-500 bg-white px-3.5 py-1.5 rounded-full border border-slate-200 shadow-2xs">
            <span class="w-2 h-2 rounded-full bg-emerald-500"></span>
            <span>Tomas a Domicilio y en Sedes</span>
          </div>
        </div>

        <div class="grid grid-cols-1 lg:grid-cols-12 gap-5 items-start">
          
          <!-- COLUMNA 1 (IZQUIERDA, 3 cols): LISTADO DE EXÁMENES -->
          <div class="lg:col-span-3 flex flex-col gap-2.5">
            <div class="text-[11px] font-black uppercase tracking-wider text-[#001D69] px-1 flex items-center justify-between">
              <span>Selecciona un Examen</span>
              <span class="text-slate-400 font-semibold">{{ servicesList.length }} disponibles</span>
            </div>

            <div 
              v-for="service in servicesList" 
              :key="service.id"
              @click="selectedService = service"
              class="p-4 rounded-2xl transition-all duration-200 cursor-pointer border text-left group"
              :class="selectedService.id === service.id 
                ? 'bg-[#001D69] text-white border-[#001D69] shadow-md translate-x-1' 
                : 'bg-white hover:bg-slate-50 text-slate-800 border-slate-200/90 shadow-2xs'"
            >
              <div class="flex items-center justify-between mb-1.5">
                <span 
                  class="text-[9px] font-black uppercase tracking-wider px-2 py-0.5 rounded"
                  :class="selectedService.id === service.id ? 'bg-[#00B4FF] text-[#001D69]' : 'bg-[#EBF7FF] text-[#004DE6]'"
                >
                  {{ service.category }}
                </span>
                <span class="text-[10px] font-bold" :class="selectedService.id === service.id ? 'text-[#00B4FF]' : 'text-slate-400'">
                  {{ service.deliveryTime }}
                </span>
              </div>

              <h4 class="font-['Outfit'] font-black text-sm leading-snug">
                {{ service.name }}
              </h4>

              <p class="text-[11px] line-clamp-1 mt-1" :class="selectedService.id === service.id ? 'text-white/70' : 'text-slate-500'">
                {{ service.shortDesc }}
              </p>
            </div>
          </div>

          <!-- COLUMNA 2 (CENTRAL, 6 cols): HOJA CLÍNICA OFICIAL TIPO INFORME -->
          <div class="lg:col-span-6 bg-white rounded-3xl p-6 sm:p-8 border border-slate-300 shadow-[0_20px_50px_rgba(0,29,105,0.06)] relative overflow-hidden font-sans">
            <!-- Marca de agua -->
            <div class="absolute inset-0 flex items-center justify-center pointer-events-none opacity-[0.035] select-none">
              <img src="/logo.png" alt="" class="w-[500px] h-[500px] object-contain" />
            </div>

            <!-- Cabecera -->
            <div class="flex items-start justify-between pb-3.5 border-b-2 border-[#001D69]">
              <div class="flex items-center gap-3.5">
                <div class="w-13 h-13 rounded-xl bg-[#001D69] p-1 flex items-center justify-center shadow-xs">
                  <img src="/logo.png" alt="Biomedilab" class="w-full h-full object-contain" />
                </div>
                <div>
                  <h3 class="font-['Outfit'] font-black text-xl text-[#001D69] tracking-tight">BIOMEDILAB</h3>
                  <p class="text-[10px] font-bold text-slate-500 uppercase tracking-widest">Laboratorio de Análisis Clínicos</p>
                </div>
              </div>
              <div class="text-right text-[10px] text-slate-500 hidden sm:block leading-tight">
                <p class="font-bold text-slate-800">Sede San Isidro: Av. Javier Prado Este 2450</p>
                <p>Telf: (01) 555-0100 • WhatsApp: +51 999 999 991</p>
                <p class="text-[#004DE6] font-semibold">www.biomedilab.pe • contacto@biomedilab.pe</p>
              </div>
            </div>

            <!-- Ficha Paciente -->
            <div class="grid grid-cols-2 sm:grid-cols-12 gap-x-3 gap-y-1.5 py-3 border-b border-slate-300 text-[11px] text-slate-800">
              <div class="sm:col-span-6 flex">
                <span class="w-20 text-slate-500 font-semibold">N. Paciente:</span>
                <span class="font-bold text-[#001D69] truncate">603574-BM</span>
              </div>
              <div class="sm:col-span-6 flex">
                <span class="w-16 text-slate-500 font-semibold">Nombre:</span>
                <span class="font-bold uppercase tracking-wide text-slate-900 truncate">RUIZ PEREJÓN, BASILIO</span>
              </div>
              <div class="sm:col-span-6 flex">
                <span class="w-20 text-slate-500 font-semibold">DNI:</span>
                <span class="font-bold">72354248</span>
              </div>
              <div class="sm:col-span-6 flex">
                <span class="w-16 text-slate-500 font-semibold">Médico Dr.:</span>
                <span class="font-bold uppercase text-slate-900 truncate">DR. FORMOSO GARCÍA</span>
              </div>
              <div class="sm:col-span-6 flex">
                <span class="w-20 text-slate-500 font-semibold">Fecha Ext.:</span>
                <span class="font-medium text-slate-700">20/10/2026 07:45 AM</span>
              </div>
              <div class="sm:col-span-6 flex">
                <span class="w-16 text-slate-500 font-semibold">Fecha Imp.:</span>
                <span class="font-medium text-slate-700">20/10/2026 11:30 AM</span>
              </div>
            </div>

            <!-- Nombre de Examen -->
            <div class="pt-4 pb-1.5 flex items-center justify-between">
              <h4 class="font-['Outfit'] font-extrabold text-base text-[#001D69] uppercase tracking-wide">
                {{ selectedService.name }}
              </h4>
              <span class="text-[9px] uppercase font-bold text-slate-400 bg-slate-100 px-2.5 py-0.5 rounded">
                {{ selectedService.category }}
              </span>
            </div>

            <!-- Cabecera Tabla -->
            <div class="border-b border-slate-300 pb-1 pt-1.5 flex items-center justify-between text-[11px] text-slate-500">
              <span class="w-1/2 font-bold uppercase text-[10px]">Determinación</span>
              <span class="w-20 text-right pr-2 italic font-serif">Resultado</span>
              <span class="w-16 text-center italic font-serif">Unidades</span>
              <span class="w-28 text-center italic font-serif">Referencia</span>
            </div>

            <!-- Analitos -->
            <div class="py-1.5 space-y-1 text-xs">
              <template v-for="(bio, bIdx) in selectedService.biomarkers" :key="bIdx">
                <div 
                  v-if="bIdx === 0 || bio.group !== selectedService.biomarkers[bIdx - 1]?.group"
                  class="pt-2.5 pb-1 text-[10px] font-black text-[#001D69] uppercase flex items-center gap-2"
                >
                  <span>{{ bio.group || 'PARÁMETROS ANALIZADOS' }}</span>
                  <span class="h-px bg-slate-200 flex-1"></span>
                </div>

                <div 
                  @click="activeBiomarkerDetail = activeBiomarkerDetail?.name === bio.name ? null : bio"
                  class="py-1 px-2 rounded-lg transition-all cursor-pointer group"
                  :class="[
                    activeBiomarkerDetail?.name === bio.name 
                      ? 'bg-[#EBF7FF] border border-[#00B4FF]/40 shadow-2xs' 
                      : bio.isHighlighted 
                        ? 'bg-amber-50/70 border border-amber-200/50 hover:bg-amber-100/50' 
                        : 'hover:bg-slate-50 border border-transparent'
                  ]"
                >
                  <div class="flex items-center justify-between text-[11px] sm:text-xs">
                    <div class="w-1/2 flex items-center gap-1.5 truncate">
                      <span v-if="bio.isHighlighted" class="text-amber-600 font-bold">*</span>
                      <span class="tracking-tight truncate" :class="bio.isHighlighted ? 'font-black text-slate-900' : 'font-medium text-slate-800'">
                        {{ bio.name }}
                      </span>
                      <span class="w-3.5 h-3.5 rounded-full bg-slate-100 text-slate-400 group-hover:bg-[#00B4FF]/20 group-hover:text-[#004DE6] flex items-center justify-center text-[8px] font-bold shrink-0 transition-colors">?</span>
                    </div>

                    <div class="w-20 text-right pr-2">
                      <span :class="bio.isHighlighted ? 'font-black text-amber-700 underline decoration-amber-400' : 'font-bold text-slate-900'">
                        {{ bio.result }}
                      </span>
                    </div>
                    <div class="w-16 text-center text-slate-500 text-[10px] sm:text-[11px]">{{ bio.unit }}</div>
                    <div class="w-28 text-center text-slate-600 font-mono text-[10px] sm:text-[11px] flex items-center justify-center gap-1">
                      <span class="text-slate-300">|</span>
                      <span>{{ bio.refRange }}</span>
                      <span class="text-slate-300">|</span>
                    </div>
                  </div>

                  <!-- Desplegable -->
                  <div v-if="activeBiomarkerDetail?.name === bio.name && bio.description" class="mt-2.5 p-3 rounded-xl bg-[#001D69] text-white text-[11px] leading-relaxed border border-[#00B4FF]/30 shadow-md">
                    <div class="flex items-center justify-between text-[#00B4FF] font-bold text-[10px] uppercase mb-1">
                      <span class="flex items-center gap-1">
                        <Info class="w-3.5 h-3.5" />
                        ¿Qué evalúa {{ bio.name }}?
                      </span>
                      <span class="text-white/60 font-normal lowercase">clic para cerrar</span>
                    </div>
                    <p class="text-white/90 text-xs mt-0.5">{{ bio.description }}</p>
                    <div class="mt-2 pt-1.5 border-t border-white/10 flex items-center justify-between text-[10px] text-white/70">
                      <span>Rango normal: <strong class="text-white">{{ bio.refRange }} {{ bio.unit }}</strong></span>
                      <span class="text-emerald-300 font-medium">Validado</span>
                    </div>
                  </div>
                </div>
              </template>
            </div>

            <!-- Nota al pie -->
            <div class="mt-4 pt-3 border-t border-slate-200 text-[11px] text-slate-600 leading-relaxed bg-slate-50/90 p-3 rounded-xl">
              <span class="font-bold text-[#001D69]">Observación Médica:</span> {{ selectedService.clinicalNote }}
            </div>
          </div>

          <!-- COLUMNA 3 (DERECHA, 3 cols): TARJETA DE PREPARACIÓN Y ACCIONES RÁPIDAS -->
          <div class="lg:col-span-3 flex flex-col gap-3 sticky top-8">
            <div class="bg-white rounded-3xl p-6 border border-slate-200/90 shadow-sm">
              <span class="text-[10px] font-black uppercase tracking-wider text-[#004DE6] bg-[#EBF7FF] px-3 py-1 rounded-md inline-block mb-3.5">
                Protocolo del Paciente
              </span>

              <h4 class="font-['Outfit'] font-black text-lg text-[#001D69] leading-snug">
                Requisitos para este Análisis
              </h4>

              <div class="space-y-3 mt-4 text-xs">
                <div class="p-3.5 rounded-2xl bg-[#F8FAFC] border border-slate-100 flex items-start gap-3">
                  <div class="w-8 h-8 rounded-xl bg-[#EBF7FF] flex items-center justify-center shrink-0">
                    <Clock class="w-4 h-4 text-[#00B4FF]" />
                  </div>
                  <div>
                    <span class="font-bold text-slate-900 block text-xs">Horas de Ayuno Requeridas</span>
                    <span class="text-[#004DE6] font-extrabold text-xs">{{ selectedService.fastingHours }}</span>
                  </div>
                </div>

                <div class="p-3.5 rounded-2xl bg-[#F8FAFC] border border-slate-100 flex items-start gap-3">
                  <div class="w-8 h-8 rounded-xl bg-emerald-50 flex items-center justify-center shrink-0">
                    <CheckCircle2 class="w-4 h-4 text-emerald-600" />
                  </div>
                  <div>
                    <span class="font-bold text-slate-900 block text-xs">Tipo de Muestra</span>
                    <span class="text-slate-600 leading-tight block mt-0.5">{{ selectedService.sampleType }}</span>
                  </div>
                </div>

                <div class="p-3.5 rounded-2xl bg-[#F8FAFC] border border-slate-100 flex items-start gap-3">
                  <div class="w-8 h-8 rounded-xl bg-blue-50 flex items-center justify-center shrink-0">
                    <Clock class="w-4 h-4 text-[#004DE6]" />
                  </div>
                  <div>
                    <span class="font-bold text-slate-900 block text-xs">Tiempo de Emisión</span>
                    <span class="text-slate-700 font-semibold">{{ selectedService.deliveryTime }}</span>
                  </div>
                </div>
              </div>

              <!-- Sedes con toma activa -->
              <div class="mt-5 pt-3.5 border-t border-slate-100 text-xs">
                <span class="text-[10px] uppercase font-bold text-slate-400 block mb-2">Disponible para toma en:</span>
                <div class="grid grid-cols-2 gap-1.5">
                  <span class="px-2.5 py-1 rounded-lg bg-slate-100 text-slate-700 text-[10px] font-semibold text-center">📍 San Isidro</span>
                  <span class="px-2.5 py-1 rounded-lg bg-slate-100 text-slate-700 text-[10px] font-semibold text-center">📍 Miraflores</span>
                  <span class="px-2.5 py-1 rounded-lg bg-slate-100 text-slate-700 text-[10px] font-semibold text-center">📍 Surco</span>
                  <span class="px-2.5 py-1 rounded-lg bg-slate-100 text-slate-700 text-[10px] font-semibold text-center">📍 Los Olivos</span>
                </div>
              </div>

              <!-- Botones de Acción -->
              <div class="space-y-2.5 mt-5">
                <a 
                  :href="`https://wa.me/51999999999?text=Hola%20Biomedilab,%20deseo%20cotizar%20y%20agendar%20${encodeURIComponent(selectedService.name)}`" 
                  target="_blank"
                  class="w-full py-3 px-4 rounded-xl bg-[#00B4FF] hover:bg-[#0092D1] text-[#001D69] font-black text-xs flex items-center justify-center gap-2 shadow-md transition-all active:scale-95"
                >
                  <MessageCircle class="w-4 h-4 text-[#001D69]" />
                  <span>Cotizar por WhatsApp</span>
                </a>

                <button 
                  @click="openModal" 
                  class="w-full py-2.5 px-4 rounded-xl bg-[#001D69] hover:bg-[#002B94] text-white font-bold text-xs flex items-center justify-center gap-2 transition-colors cursor-pointer"
                >
                  <Download class="w-4 h-4 text-[#00B4FF]" />
                  <span>Descargar Plantilla PDF</span>
                </button>
              </div>

            </div>
          </div>

        </div>
      </div>
    </section>

    <!-- ========================================================================= -->
    <!-- SECCIÓN 4: NUESTRAS SEDES CON GOOGLE MAPS INMERSIVO Y BORDES REDONDEADOS -->
    <!-- ========================================================================= -->
    <section id="sedes-mapa" class="relative w-full bg-[#001D69] text-white py-20 overflow-hidden">
      
      <div class="absolute -top-24 right-0 w-96 h-96 bg-[#00B4FF]/10 rounded-full blur-3xl pointer-events-none"></div>

      <div class="max-w-[96%] xl:max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 mb-6">
        <div class="flex flex-col md:flex-row md:items-end justify-between gap-4">
          <div>
            <div class="inline-flex items-center gap-2 px-3 py-1 rounded-full bg-white/10 border border-white/20 text-[#00B4FF] text-xs font-bold uppercase tracking-wider mb-2">
              <Compass class="w-3.5 h-3.5" />
              <span>Red de Atención Clínica en Lima</span>
            </div>
            <h2 class="font-['Outfit'] font-black text-2xl sm:text-3xl lg:text-4xl text-white tracking-tight">
              Encuentra tu Sede Biomedilab más Cercana
            </h2>
            <p class="text-xs sm:text-sm text-white/70 mt-1 max-w-xl">
              Selecciona cualquiera de nuestras 4 sedes en el selector lateral del mapa para centrar la ubicación con precisión y acceder al agendamiento directo.
            </p>
          </div>

          <div class="flex items-center gap-2 text-xs text-white/70">
            <span class="w-2.5 h-2.5 rounded-full bg-emerald-400 animate-pulse"></span>
            <span>4 Sedes con Tomas de Muestra en Ayunas</span>
          </div>
        </div>
      </div>

      <!-- CONTENEDOR DEL MAPA CON BORDES REDONDEADOS: MÁS ANCHO Y MENOS ALTURA CON TRANSICIÓN SUAVE -->
      <div class="max-w-[96%] xl:max-w-[1440px] mx-auto px-2 sm:px-4">
        <div class="relative w-full h-[520px] rounded-3xl overflow-hidden border-2 border-white/20 shadow-[0_25px_60px_rgba(0,0,0,0.45)] bg-slate-900">
          
          <!-- CONTENEDOR LEAFLET INTERACTIVO CON FLYTO FLUIDO SIN PARPADEO -->
          <div ref="mapContainer" class="w-full h-full z-0"></div>

          <!-- PANEL IZQUIERDO INTERNO: INFORMACIÓN DE LA SEDE, SEPARAR CITA Y WHATSAPP -->
          <div class="absolute bottom-6 left-6 top-auto sm:top-6 sm:bottom-auto z-20 w-[calc(100%-3rem)] sm:w-84">
            <div class="bg-[#001D69]/90 backdrop-blur-xl border border-white/25 rounded-3xl p-5 shadow-[0_20px_50px_rgba(0,0,0,0.6)] text-white">
              
              <div class="flex items-center justify-between gap-2 mb-2.5">
                <span class="px-2.5 py-0.5 rounded-md text-[10px] font-black uppercase tracking-wider bg-[#00B4FF] text-[#001D69]">
                  {{ selectedBranch.tag }}
                </span>
                <span class="text-[11px] text-emerald-400 font-bold flex items-center gap-1">
                  <span class="w-2 h-2 rounded-full bg-emerald-400 animate-pulse"></span>
                  Abierto
                </span>
              </div>

              <h3 class="font-['Outfit'] font-black text-lg text-white leading-tight">
                {{ selectedBranch.name }}
              </h3>
              
              <p class="text-[11px] text-white/80 flex items-start gap-1.5 mt-2 leading-relaxed">
                <MapPin class="w-3.5 h-3.5 text-[#00B4FF] shrink-0 mt-0.5" />
                <span>{{ selectedBranch.address }}</span>
              </p>

              <p class="text-[10px] text-white/60 ml-5">
                {{ selectedBranch.reference }}
              </p>

              <div class="mt-3 pt-2.5 border-t border-white/15 space-y-1.5 text-[11px]">
                <div class="flex items-center gap-2 text-white/90">
                  <Phone class="w-3.5 h-3.5 text-[#00B4FF] shrink-0" />
                  <span>Central: {{ selectedBranch.phone }}</span>
                </div>
                <div class="flex items-center gap-2 text-white/90">
                  <Clock class="w-3.5 h-3.5 text-[#00B4FF] shrink-0" />
                  <span>{{ selectedBranch.hours }}</span>
                </div>
                <div class="flex items-center gap-2 text-amber-300 font-semibold text-[10px]">
                  <CheckCircle2 class="w-3 h-3 shrink-0" />
                  <span>{{ selectedBranch.fastingHours }}</span>
                </div>
              </div>

              <!-- BOTONES DE ACCIÓN: SEPARAR CITA, WHATSAPP Y CÓMO LLEGAR -->
              <div class="space-y-2 mt-4 pt-1">
                <!-- Botón Separar Cita -->
                <button 
                  @click="openModal"
                  class="w-full py-2.5 px-3 rounded-xl bg-gradient-to-r from-[#00B4FF] to-[#0092D1] hover:brightness-110 text-[#001D69] font-black text-xs flex items-center justify-center gap-2 transition-all shadow-[0_4px_15px_rgba(0,180,255,0.4)] cursor-pointer active:scale-95"
                >
                  <Calendar class="w-4 h-4" />
                  <span>Separar Cita en esta Sede</span>
                </button>

                <!-- Fila doble: WhatsApp y Cómo Llegar -->
                <div class="grid grid-cols-2 gap-2">
                  <a 
                    :href="`https://wa.me/${selectedBranch.whatsapp}?text=Hola%20Biomedilab,%20deseo%20atenci%C3%B3n%20para%20la%20sede%20${encodeURIComponent(selectedBranch.name)}`" 
                    target="_blank"
                    class="py-2 px-2.5 rounded-xl bg-white/10 hover:bg-emerald-600 border border-white/20 text-white font-bold text-[11px] flex items-center justify-center gap-1.5 transition-all text-center"
                  >
                    <MessageCircle class="w-3.5 h-3.5 text-emerald-400 group-hover:text-white" />
                    <span>WhatsApp</span>
                  </a>

                  <a 
                    :href="selectedBranch.mapsDirectionUrl" 
                    target="_blank"
                    class="py-2 px-2.5 rounded-xl bg-white/10 hover:bg-white/20 border border-white/20 text-white font-bold text-[11px] flex items-center justify-center gap-1.5 transition-all text-center"
                  >
                    <Navigation class="w-3.5 h-3.5 text-[#00B4FF]" />
                    <span>Cómo Llegar</span>
                  </a>
                </div>
              </div>

            </div>
          </div>

          <!-- PANEL DERECHO INTERNO: SELECTOR DE LAS 4 SEDES DENTRO DEL MAPA -->
          <div class="absolute top-6 right-6 z-20 hidden md:flex flex-col gap-2 w-72">
            <div class="bg-[#001D69]/85 backdrop-blur-xl border border-white/20 rounded-2xl p-2.5 shadow-[0_15px_40px_rgba(0,0,0,0.5)]">
              <div class="text-[10px] uppercase font-black text-[#00B4FF] px-2 py-1 tracking-wider flex items-center justify-between">
                <span>Nuestras 4 Sedes</span>
                <span class="text-white/60 font-normal">Clic para centrar</span>
              </div>

              <div class="space-y-1.5 mt-1">
                <div 
                  v-for="b in branches" 
                  :key="b.id"
                  @click="selectBranch(b)"
                  class="p-2.5 rounded-xl border transition-all cursor-pointer flex items-center justify-between group"
                  :class="selectedBranch.id === b.id 
                    ? 'bg-[#002B94] border-[#00B4FF] shadow-[0_0_15px_rgba(0,180,255,0.4)] translate-x-[-2px]' 
                    : 'bg-white/5 border-white/10 hover:bg-white/15'"
                >
                  <div class="pr-2 truncate">
                    <div class="flex items-center gap-1.5">
                      <span 
                        class="w-2 h-2 rounded-full"
                        :class="selectedBranch.id === b.id ? 'bg-[#00B4FF] shadow-[0_0_6px_#00B4FF]' : 'bg-white/40'"
                      ></span>
                      <h4 class="font-['Outfit'] font-bold text-xs text-white group-hover:text-[#00B4FF] transition-colors truncate">
                        {{ b.district }}
                      </h4>
                    </div>
                    <p class="text-[10px] text-white/60 truncate pl-3.5 mt-0.5">{{ b.address }}</p>
                  </div>

                  <span 
                    class="text-[10px] font-bold px-2 py-0.5 rounded-md shrink-0 transition-colors"
                    :class="selectedBranch.id === b.id ? 'bg-[#00B4FF] text-[#001D69]' : 'text-[#00B4FF] group-hover:bg-white/10'"
                  >
                    Ver
                  </span>
                </div>
              </div>
            </div>
          </div>

          <!-- SELECTOR DE SEDES INFERIOR PARA MÓVILES (cuando la pantalla es pequeña) -->
          <div class="absolute top-4 right-4 z-20 flex md:hidden gap-1.5 overflow-x-auto max-w-[calc(100%-2rem)] bg-[#001D69]/90 p-1 rounded-xl border border-white/20 backdrop-blur-md">
            <button 
              v-for="b in branches" 
              :key="b.id"
              @click="selectBranch(b)"
              class="px-2.5 py-1 rounded-lg text-[10px] font-bold whitespace-nowrap cursor-pointer"
              :class="selectedBranch.id === b.id ? 'bg-[#00B4FF] text-[#001D69]' : 'text-white/80'"
            >
              {{ b.district }}
            </button>
          </div>

        </div>
      </div>

    </section>

    <!-- ================= MODAL DE RESULTADOS ================= -->
    <div 
      v-if="isModalOpen" 
      class="fixed inset-0 z-50 bg-slate-950/80 backdrop-blur-md flex items-center justify-center p-4 animate-in fade-in duration-200"
    >
      <div class="bg-[#002277] border border-white/30 rounded-3xl max-w-md w-full p-6 sm:p-8 shadow-2xl relative text-white">
        <button @click="closeModal" class="absolute top-5 right-5 text-white/60 hover:text-white p-1 rounded-full hover:bg-white/10 transition-colors">
          <X class="w-5 h-5" />
        </button>

        <div class="text-center mb-6">
          <div class="w-12 h-12 rounded-2xl bg-[#00B4FF] text-[#002B94] flex items-center justify-center mx-auto mb-3 shadow-[0_0_20px_rgba(0,180,255,0.4)]">
            <FileCheck2 class="w-6 h-6" />
          </div>
          <h3 class="font-['Outfit'] font-black text-2xl text-white">Consulta tus Resultados</h3>
          <p class="text-xs text-white/70 mt-1">Ingresa los datos impresos en tu comprobante de laboratorio.</p>
        </div>

        <form @submit="handleResultsSubmit" class="space-y-4">
          <div>
            <label class="block text-xs font-bold text-white/90 mb-1">DNI o Documento de Identidad</label>
            <input type="text" required placeholder="Ej. 72485912" class="w-full px-4 py-2.5 rounded-xl bg-white/10 border border-white/20 text-sm text-white placeholder-white/40 focus:outline-none focus:border-[#00B4FF] focus:ring-1 focus:ring-[#00B4FF]" />
          </div>

          <div>
            <label class="block text-xs font-bold text-white/90 mb-1">Código de Orden / Ticket</label>
            <input type="text" required placeholder="Ej. BIO-8492" class="w-full px-4 py-2.5 rounded-xl bg-white/10 border border-white/20 text-sm text-white uppercase placeholder-white/40 focus:outline-none focus:border-[#00B4FF] focus:ring-1 focus:ring-[#00B4FF]" />
          </div>

          <button type="submit" class="w-full py-3 rounded-xl bg-[#00B4FF] hover:bg-white text-[#002B94] font-black text-xs shadow-[0_0_25px_rgba(0,180,255,0.5)] transition-all flex items-center justify-center gap-2 mt-2">
            <Download class="w-4 h-4" />
            <span>Descargar Informe Clínico en PDF</span>
          </button>
        </form>

        <p class="text-[11px] text-white/50 text-center mt-4">
          ¿No tienes tu código? <a href="https://wa.me/51999999999" target="_blank" class="text-[#00B4FF] font-semibold hover:underline">Solicítalo por WhatsApp</a>
        </p>
      </div>
    </div>

    <!-- ========================================================================= -->
    <!-- BOTÓN FLOTANTE INFERIOR DERECHO CON MICROSCOPIO Y BORDE CIRCULAR DE PROGRESO -->
    <!-- ========================================================================= -->
    <div 
      class="fixed bottom-6 right-6 z-40 transition-all duration-500 ease-out"
      :class="scrollProgress > 3 ? 'opacity-100 translate-y-0 scale-100' : 'opacity-0 translate-y-8 scale-75 pointer-events-none'"
    >
      <button 
        @click="scrollToTop"
        class="relative w-14 h-14 rounded-full flex items-center justify-center bg-[#001D69] text-white shadow-[0_10px_30px_rgba(0,15,50,0.6)] hover:shadow-[0_10px_35px_rgba(0,180,255,0.7)] hover:scale-110 active:scale-95 transition-all duration-300 group cursor-pointer"
        title="Volver al inicio"
        aria-label="Volver arriba"
      >
        <!-- Anillo SVG de progreso circular -->
        <svg class="absolute inset-0 w-full h-full -rotate-90 pointer-events-none" viewBox="0 0 56 56">
          <!-- Círculo de fondo tenue -->
          <circle 
            cx="28" 
            cy="28" 
            r="24" 
            stroke="rgba(255,255,255,0.15)" 
            stroke-width="3" 
            fill="none" 
          />
          <!-- Círculo de progreso animado que se llena con el scroll -->
          <circle 
            cx="28" 
            cy="28" 
            r="24" 
            stroke="#00B4FF" 
            stroke-width="3" 
            stroke-linecap="round"
            fill="none" 
            :stroke-dasharray="150.8" 
            :stroke-dashoffset="150.8 - (150.8 * scrollProgress) / 100" 
            class="transition-all duration-150 ease-out"
          />
        </svg>

        <!-- Icono de Microscopio con micro-interacción -->
        <div class="relative z-10 flex flex-col items-center justify-center">
          <Microscope class="w-6 h-6 text-[#00B4FF] group-hover:text-white transition-colors duration-300 transform group-hover:-translate-y-0.5" />
        </div>

        <!-- Tooltip flotante al pasar el mouse -->
        <span class="absolute right-16 px-2.5 py-1 rounded-lg bg-[#001D69] text-white text-[10px] font-bold border border-white/20 whitespace-nowrap opacity-0 group-hover:opacity-100 transition-opacity pointer-events-none shadow-md">
          Subir al inicio ({{ scrollProgress }}%)
        </span>
      </button>
    </div>

  </div>
</template>

<style scoped>
.no-scrollbar::-webkit-scrollbar {
  display: none;
}
.no-scrollbar {
  -ms-overflow-style: none;
  scrollbar-width: none;
}
</style>
