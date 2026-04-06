import streamlit as st
import pandas as pd
from typing import Dict, List, Tuple

# ============================================================================
# DATABASE: All diseases with symptoms, species, and metadata
# ============================================================================

diseases_db = [
    # CATTLE
    {"disease": "Bovine Respiratory Disease (BRD)", "species": ["Cattle"], "age_risk": ["Juvenile", "Adult"], "symptoms": {"fever":1,"nasal_discharge":1,"cough":1,"anorexia":1,"respiratory_distress":1}, "etiology": "*Mannheimia haemolytica, Pasteurella multocida, Histophilus somni, Mycoplasma bovis*", "transmission": "Inhalation, direct contact, fomites, stress", "prevention": "Vaccination, stress reduction, ventilation", "treatment": "Tulathromycin (DoC), florfenicol, ceftiofur"},
    {"disease": "Neonatal Calf Diarrhea (E. coli)", "species": ["Cattle"], "age_risk": ["Neonate"], "symptoms": {"fever":1,"diarrhea":1,"hemorrhagic_diarrhea":1,"anorexia":1}, "etiology": "*Escherichia coli* (K99/F5)", "transmission": "Fecal-oral, contaminated colostrum/milk", "prevention": "Adequate colostrum, vaccination of dams, sanitation", "treatment": "Ceftiofur (for septicemia), fluids"},
    {"disease": "Salmonellosis (Bovine)", "species": ["Cattle"], "age_risk": ["Neonate","Juvenile"], "symptoms": {"fever":1,"diarrhea":1,"hemorrhagic_diarrhea":1,"anorexia":1,"abortion":1,"sudden_death":1}, "etiology": "*Salmonella* Typhimurium, Dublin", "transmission": "Fecal-oral, contaminated feed/water", "prevention": "Biosecurity, test-and-cull", "treatment": "Ceftiofur, enrofloxacin"},
    {"disease": "Johne's Disease", "species": ["Cattle"], "age_risk": ["Adult"], "symptoms": {"diarrhea":1,"weight_loss":1}, "etiology": "*Mycobacterium avium* subsp. *paratuberculosis*", "transmission": "Fecal-oral, colostrum/milk", "prevention": "Test-and-cull, biosecurity", "treatment": "No effective treatment (cull)"},
    {"disease": "Blackleg", "species": ["Cattle"], "age_risk": ["Juvenile","Adult"], "symptoms": {"fever":1,"anorexia":1,"lameness":1,"skin_lesions":1,"respiratory_distress":1,"sudden_death":1}, "etiology": "*Clostridium chauvoei*", "transmission": "Spores in soil, wound contamination", "prevention": "Vaccination (multivalent clostridial)", "treatment": "Procaine Penicillin G (high dose)"},
    {"disease": "Enterotoxemia (C. perfringens)", "species": ["Cattle","Goat","Sheep"], "age_risk": ["Juvenile","Adult"], "symptoms": {"diarrhea":1,"anorexia":1,"neurologic_signs":1,"sudden_death":1}, "etiology": "*Clostridium perfringens* types C & D", "transmission": "Ingestion of spores, feed changes", "prevention": "Vaccination, avoid sudden feed changes", "treatment": "Procaine Penicillin G"},
    {"disease": "Bovine Mastitis (clinical)", "species": ["Cattle"], "age_risk": ["Adult"], "symptoms": {"fever":1,"anorexia":1,"mastitis":1}, "etiology": "*Staph. aureus, Strep. agalactiae, E. coli, Klebsiella, Mycoplasma*", "transmission": "Environmental, milking equipment, fomites", "prevention": "Teat dipping, dry cow therapy", "treatment": "Intramammary cephapirin; systemic ceftiofur"},
    # GOAT & SHEEP
    {"disease": "Pneumonia (Mannheimia/Pasteurella)", "species": ["Goat","Sheep"], "age_risk": ["Juvenile","Adult"], "symptoms": {"fever":1,"nasal_discharge":1,"cough":1,"anorexia":1,"respiratory_distress":1}, "etiology": "*Mannheimia haemolytica, Pasteurella multocida*", "transmission": "Inhalation, stress", "prevention": "Vaccination, housing", "treatment": "Tulathromycin"},
    {"disease": "Caseous Lymphadenitis (CL)", "species": ["Goat","Sheep"], "age_risk": ["Adult"], "symptoms": {"weight_loss":1,"abscesses":1}, "etiology": "*Corynebacterium pseudotuberculosis*", "transmission": "Direct contact with pus, shearing equipment", "prevention": "Culling, vaccination", "treatment": "Penicillin or tulathromycin (partial)"},
    {"disease": "Foot Rot", "species": ["Goat","Sheep"], "age_risk": ["Juvenile","Adult"], "symptoms": {"lameness":1,"foot_lesions":1}, "etiology": "*Dichelobacter nodosus, Fusobacterium necrophorum*", "transmission": "Direct contact, contaminated ground", "prevention": "Vaccination, footbaths, culling", "treatment": "Long-acting oxytetracycline or ceftiofur"},
    # HORSE
    {"disease": "Strangles", "species": ["Horse"], "age_risk": ["Juvenile","Adult"], "symptoms": {"fever":1,"nasal_discharge":1,"cough":1,"anorexia":1,"lymphadenopathy":1}, "etiology": "*Streptococcus equi* subsp. *equi*", "transmission": "Direct contact, fomites, aerosol", "prevention": "Vaccination, quarantine", "treatment": "Procaine Penicillin G (complicated cases)"},
    {"disease": "Equine Salmonellosis", "species": ["Horse"], "age_risk": ["Adult"], "symptoms": {"fever":1,"diarrhea":1,"hemorrhagic_diarrhea":1,"colic":1,"abortion":1}, "etiology": "*Salmonella enterica* (Typhimurium, Newport, Anatum)", "transmission": "Fecal-oral, stress, contaminated environment", "prevention": "Biosecurity, hygiene", "treatment": "Gentamicin + ceftiofur (septicemia only)"},
    {"disease": "Dermatophilosis (Rain Scald)", "species": ["Horse"], "age_risk": ["Juvenile","Adult"], "symptoms": {"skin_lesions":1}, "etiology": "*Dermatophilus congolensis*", "transmission": "Direct contact, fomites, prolonged wetting", "prevention": "Keep dry, individual tack", "treatment": "Oxytetracycline or procaine penicillin G"},
    # SWINE
    {"disease": "Swine Erysipelas", "species": ["Swine"], "age_risk": ["Juvenile","Adult"], "symptoms": {"fever":1,"anorexia":1,"skin_lesions":1,"lameness":1,"sudden_death":1,"abortion":1}, "etiology": "*Erysipelothrix rhusiopathiae*", "transmission": "Oral, skin abrasions, carrier pigs", "prevention": "Vaccination", "treatment": "Penicillin"},
    {"disease": "Swine Dysentery", "species": ["Swine"], "age_risk": ["Juvenile","Adult"], "symptoms": {"fever":1,"diarrhea":1,"hemorrhagic_diarrhea":1,"anorexia":1,"weight_loss":1}, "etiology": "*Brachyspira hyodysenteriae*", "transmission": "Fecal-oral, fomites, rodents", "prevention": "All-in/all-out, biosecurity, rodent control", "treatment": "Tiamulin (in water)"},
    {"disease": "Glasser's Disease", "species": ["Swine"], "age_risk": ["Juvenile"], "symptoms": {"fever":1,"anorexia":1,"lameness":1,"respiratory_distress":1,"neurologic_signs":1,"sudden_death":1}, "etiology": "*Glaesserella parasuis*", "transmission": "Oronasal, direct contact, carrier sows", "prevention": "Vaccination, all-in/all-out", "treatment": "Ceftiofur"},
    {"disease": "Porcine Proliferative Enteropathy (PPE)", "species": ["Swine"], "age_risk": ["Juvenile","Adult"], "symptoms": {"diarrhea":1,"hemorrhagic_diarrhea":1,"anorexia":1,"weight_loss":1}, "etiology": "*Lawsonia intracellularis*", "transmission": "Fecal-oral", "prevention": "Vaccination (oral), feed antimicrobials", "treatment": "Tiamulin, tylosin (in feed/water)"},
    # DOG
    {"disease": "Kennel Cough (CIRDC)", "species": ["Dog"], "age_risk": ["Juvenile","Adult"], "symptoms": {"fever":1,"cough":1,"nasal_discharge":1,"lethargy":1}, "etiology": "*Bordetella bronchiseptica*", "transmission": "Aerosol, direct contact, fomites", "prevention": "Vaccination (intranasal/oral)", "treatment": "Doxycycline"},
    {"disease": "Leptospirosis (Canine)", "species": ["Dog"], "age_risk": ["Juvenile","Adult"], "symptoms": {"fever":1,"vomiting":1,"diarrhea":1,"anorexia":1,"lethargy":1,"jaundice":1,"lymphadenopathy":1,"uveitis":1}, "etiology": "*Leptospira interrogans* serovars", "transmission": "Contact with urine of wildlife/rodents, contaminated water", "prevention": "Vaccination, rodent control", "treatment": "Doxycycline (oral), ampicillin (parenteral initial)"},
    {"disease": "Staphylococcal Pyoderma", "species": ["Dog"], "age_risk": ["Juvenile","Adult"], "symptoms": {"skin_lesions":1}, "etiology": "*Staphylococcus pseudintermedius*", "transmission": "Opportunistic (allergy, endocrinopathy)", "prevention": "Treat underlying cause, chlorhexidine shampoos", "treatment": "Cephalexin"},
    # CAT
    {"disease": "Tularemia (Feline)", "species": ["Cat"], "age_risk": ["Juvenile","Adult"], "symptoms": {"fever":1,"anorexia":1,"lethargy":1,"icterus":1,"lymphadenopathy":1,"skin_ulcers":1}, "etiology": "*Francisella tularensis*", "transmission": "Tick bite, ingestion of infected prey, direct contact", "prevention": "Tick control, prevent hunting", "treatment": "Gentamicin"},
    {"disease": "Feline Mycoplasmosis (FIA)", "species": ["Cat"], "age_risk": ["Juvenile","Adult"], "symptoms": {"fever":1,"anorexia":1,"lethargy":1,"pale_mucous_membranes":1,"icterus":1}, "etiology": "*Mycoplasma haemofelis*", "transmission": "Bite wounds, blood transfusion, fleas/ticks", "prevention": "Flea/tick control, prevent fighting", "treatment": "Doxycycline"},
    {"disease": "Feline Respiratory Disease (Snuffles)", "species": ["Cat"], "age_risk": ["Juvenile","Adult"], "symptoms": {"fever":1,"anorexia":1,"lethargy":1,"conjunctivitis":1,"nasal_discharge":1,"sneezing":1}, "etiology": "*Chlamydia felis, Bordetella bronchiseptica, Mycoplasma*", "transmission": "Direct contact (ocular/nasal), aerosol", "prevention": "Vaccination (Chlamydia, Bordetella)", "treatment": "Doxycycline"},
    # POULTRY
    {"disease": "Pullorum Disease", "species": ["Poultry"], "age_risk": ["Neonate","Juvenile"], "symptoms": {"fever":1,"whitish_diarrhea":1,"sudden_death":1}, "etiology": "*Salmonella Pullorum*", "transmission": "Vertical (transovarian), fecal-oral", "prevention": "Eradication (testing/culling)", "treatment": "Not recommended (cull)"},
    {"disease": "Fowl Cholera", "species": ["Poultry"], "age_risk": ["Juvenile","Adult"], "symptoms": {"fever":1,"nasal_discharge":1,"facial_swelling":1,"diarrhea":1,"cyanosis":1,"sudden_death":1,"drop_in_egg_production":1,"joint_swelling":1}, "etiology": "*Pasteurella multocida*", "transmission": "Oral/nasal, contaminated water, rodents", "prevention": "Vaccination, rodent control", "treatment": "Penicillin (in water)"},
    {"disease": "Infectious Coryza", "species": ["Poultry"], "age_risk": ["Juvenile","Adult"], "symptoms": {"fever":1,"nasal_discharge":1,"sneezing":1,"facial_swelling":1,"drop_in_egg_production":1}, "etiology": "*Avibacterium paragallinarum*", "transmission": "Direct contact, aerosol, fomites", "prevention": "Vaccination (bacterins), all-in/all-out", "treatment": "Tylosin (in water)"},
    {"disease": "Necrotic Enteritis", "species": ["Poultry"], "age_risk": ["Juvenile","Adult"], "symptoms": {"diarrhea":1,"sudden_death":1,"enteritis":1}, "etiology": "*Clostridium perfringens*", "transmission": "Opportunistic (coccidial damage, diet)", "prevention": "Coccidiosis control, feed management", "treatment": "Penicillin (in feed/water)"},
]

# Normalize symptom names for user interface
all_possible_symptoms = sorted(set(
    s for disease in diseases_db for s in disease["symptoms"].keys()
))

# Map user-friendly labels to internal keys
symptom_labels = {
    "fever": "Fever (≥40°C / 104°F)",
    "nasal_discharge": "Nasal Discharge",
    "cough": "Cough",
    "diarrhea": "Diarrhea (watery)",
    "hemorrhagic_diarrhea": "Hemorrhagic Diarrhea (bloody)",
    "anorexia": "Anorexia / Reduced Appetite",
    "weight_loss": "Weight Loss (chronic)",
    "lameness": "Lameness",
    "skin_lesions": "Skin Lesions (pustules, crusts, ulcers)",
    "abscesses": "Abscesses (lymph nodes or body)",
    "respiratory_distress": "Respiratory Distress (dyspnea, tachypnea)",
    "neurologic_signs": "Neurologic Signs (circling, tremors, seizures)",
    "sudden_death": "Sudden Death (acute)",
    "swollen_joints": "Swollen Joints",
    "mastitis": "Mastitis (swollen, painful udder)",
    "abortion": "Abortion / Stillbirth",
    "colic": "Colic (abdominal pain)",
    "lymphadenopathy": "Enlarged Lymph Nodes",
    "jaundice": "Jaundice (icterus)",
    "uveitis": "Uveitis (eye inflammation)",
    "conjunctivitis": "Conjunctivitis (eye redness/discharge)",
    "sneezing": "Sneezing",
    "facial_swelling": "Facial Swelling (periorbital, wattles)",
    "cyanosis": "Cyanosis (blue/purple combs or mucous membranes)",
    "drop_in_egg_production": "Drop in Egg Production",
    "whitish_diarrhea": "Whitish Diarrhea (pasted vent in poultry)",
    "enteritis": "Enteritis (inflammation of intestines)",
    "vomiting": "Vomiting",
    "lethargy": "Lethargy / Depression",
    "pale_mucous_membranes": "Pale Mucous Membranes (anemia)",
    "skin_ulcers": "Skin Ulcers",
    "foot_lesions": "Foot Lesions (necrosis, foul odor)",
}

# Age risk mapping
age_groups = ["Neonate (0-4 weeks)", "Juvenile (weaning to 6 months)", "Adult (mature)", "Geriatric (senior)"]
age_to_risk = {
    "Neonate (0-4 weeks)": "Neonate",
    "Juvenile (weaning to 6 months)": "Juvenile",
    "Adult (mature)": "Adult",
    "Geriatric (senior)": "Adult"  # geriatric treated as adult risk
}

# ============================================================================
# SCORING FUNCTION
# ============================================================================

def score_disease(disease: dict, user_symptoms: Dict[str, bool], species: str, age_risk_key: str) -> float:
    """Returns a match score (0-100) for a disease given user inputs."""
    # 1. Species match
    if species not in disease["species"]:
        return 0.0
    # 2. Age match (if disease specifies age_risk)
    if "age_risk" in disease and disease["age_risk"]:
        if age_risk_key not in disease["age_risk"]:
            return 0.0
    # 3. Symptom matching
    disease_symptoms = disease["symptoms"]
    total_symptoms_in_disease = len(disease_symptoms)
    if total_symptoms_in_disease == 0:
        return 0.0
    matched = 0
    for sym, present in user_symptoms.items():
        if present and sym in disease_symptoms:
            matched += 1
    # Optional: penalize if user reported a symptom that is NOT in disease (not implemented for simplicity)
    score = (matched / total_symptoms_in_disease) * 100
    return round(score, 1)

# ============================================================================
# STREAMLIT UI
# ============================================================================

st.set_page_config(page_title="VetDx - Bacterial Disease Decision Support", layout="wide")
st.title("🩺 VetDx: Veterinary Bacterial Disease Assistant")
st.markdown("**For professional use only** – This tool suggests differential diagnoses based on clinical signs. Always confirm with appropriate diagnostics.")

# Sidebar for patient information
with st.sidebar:
    st.header("🐾 Patient Information")
    species = st.selectbox("Species", ["Cattle", "Goat", "Sheep", "Horse", "Swine", "Dog", "Cat", "Poultry"])
    age = st.selectbox("Age category", age_groups)
    sex = st.radio("Sex", ["Male (intact)", "Male (neutered)", "Female (intact)", "Female (spayed)"], horizontal=True)
    vaccinated = st.radio("Vaccination status (relevant for clostridial, leptospirosis, etc.)", ["Unknown", "Up-to-date", "Not vaccinated"], index=0)
    st.markdown("---")
    st.caption("Disclaimer: This app provides decision support based on a limited database. It does not replace clinical judgment or laboratory confirmation.")

# Main area: symptom selection
st.subheader("📋 Select Presenting Clinical Signs")
cols = st.columns(3)
symptom_selection = {}
for i, (sym_key, sym_label) in enumerate(symptom_labels.items()):
    with cols[i % 3]:
        symptom_selection[sym_key] = st.checkbox(sym_label, value=False)

# Add optional free-text notes
st.text_area("Additional notes (e.g., environment, herd history, prior treatments)", height=68)

# Diagnosis button
if st.button("🔍 Generate Differential Diagnoses", type="primary"):
    with st.spinner("Analyzing symptoms..."):
        age_risk_key = age_to_risk[age]
        user_symptoms = {k: v for k, v in symptom_selection.items() if v}
        if not user_symptoms:
            st.warning("Please select at least one clinical sign to proceed.")
        else:
            scored = []
            for disease in diseases_db:
                score = score_disease(disease, user_symptoms, species, age_risk_key)
                if score > 0:
                    scored.append((score, disease))
            scored.sort(key=lambda x: x[0], reverse=True)
            if not scored:
                st.error("No matching bacterial diseases found for the given signs and species. Consider non-bacterial causes or review symptoms.")
            else:
                st.success(f"Found {len(scored)} possible differentials. Top results below:")
                for rank, (score, d) in enumerate(scored[:5], 1):
                    with st.expander(f"#{rank}: {d['disease']}  (Match: {score}%)"):
                        col1, col2 = st.columns(2)
                        with col1:
                            st.markdown(f"**📖 Etiology**  \n{d['etiology']}")
                            st.markdown(f"**🔄 Transmission**  \n{d['transmission']}")
                        with col2:
                            st.markdown(f"**🛡️ Prevention**  \n{d['prevention']}")
                            st.markdown(f"**💊 Treatment (Drug of Choice)**  \n{d['treatment']}")
                        # Additional: show which symptoms matched
                        matched_symptoms = [symptom_labels[s] for s in d["symptoms"] if user_symptoms.get(s, False)]
                        if matched_symptoms:
                            st.markdown(f"**✅ Present signs that support this diagnosis:** {', '.join(matched_symptoms)}")
                st.info("💡 **Clinical tip**: For a definitive diagnosis, consider culture, PCR, or serology. Always follow local antimicrobial stewardship guidelines.")
