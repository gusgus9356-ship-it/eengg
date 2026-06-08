import streamlit as st
import pandas as pd
import random, json, os, hashlib, re
from datetime import datetime, date
try:
    from pypdf import PdfReader
    PDF_OK = True
except ImportError:
    PDF_OK = False

st.set_page_config(page_title="영어 암기 스튜디오", page_icon="📖", layout="wide", initial_sidebar_state="expanded")

# ── CSS ──────────────────────────────────────────────────────────────
st.markdown("""
<style>
@import url('https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@300;400;500;700&family=Inter:wght@400;500;600&display=swap');
html,body,[class*="css"]{font-family:'Noto Sans KR',sans-serif;}
.stApp{background:#F0F4F8;}
section[data-testid="stSidebar"]{background:#FFFFFF!important;border-right:1px solid #E2E8F0;}
section[data-testid="stSidebar"] *{color:#374151!important;}

/* 타이포 */
.page-title{font-size:1.5rem;font-weight:700;color:#1E3A5F;margin-bottom:2px;}
.page-sub{font-size:0.82rem;color:#6B7280;margin-bottom:20px;letter-spacing:.3px;}

/* 사이드바 */
.sb-logo{padding:0 16px 14px;border-bottom:1px solid #E2E8F0;margin-bottom:10px;}
.sb-logo-icon{width:36px;height:36px;border-radius:10px;background:linear-gradient(135deg,#1E3A8A,#2563EB);display:flex;align-items:center;justify-content:center;margin-bottom:8px;}
.sb-app-name{font-size:13px;font-weight:700;color:#1E3A5F!important;}
.sb-app-sub{font-size:10px;color:#9CA3AF!important;}
.sb-user{margin:0 10px 12px;background:#EFF6FF;border-radius:10px;padding:9px 12px;display:flex;align-items:center;gap:9px;}
.sb-avatar{width:28px;height:28px;border-radius:50%;background:#2563EB;color:#fff;font-size:11px;font-weight:600;display:flex;align-items:center;justify-content:center;}
.sb-name{font-size:12px;font-weight:600;color:#1E3A8A!important;}
.sb-role{font-size:10px;color:#93C5FD!important;}
.sb-section{font-size:10px;color:#9CA3AF!important;padding:8px 16px 3px;letter-spacing:.8px;text-transform:uppercase;}
.sb-filter{margin:6px 10px;padding:8px 12px;border:1px solid #E2E8F0;border-radius:8px;background:#F8FAFC;}
.sb-filter-label{font-size:10px;color:#9CA3AF!important;margin-bottom:3px;}

/* 통계 카드 */
.stat-card{background:#fff;border:1px solid #E2E8F0;border-radius:14px;padding:16px;position:relative;overflow:hidden;}
.stat-icon{width:36px;height:36px;border-radius:10px;display:flex;align-items:center;justify-content:center;font-size:17px;margin-bottom:10px;}
.si-blue{background:#DBEAFE;color:#1D4ED8;}
.si-green{background:#DCFCE7;color:#15803D;}
.si-amber{background:#FEF9C3;color:#A16207;}
.si-teal{background:#CCFBF1;color:#0F766E;}
.si-rose{background:#FFE4E6;color:#BE123C;}
.stat-val{font-size:1.6rem;font-weight:700;color:#1E293B;line-height:1;}
.stat-label{font-size:11px;color:#6B7280;margin-top:4px;}
.stat-trend{font-size:11px;color:#15803D;margin-top:5px;}

/* 패널 */
.panel{background:#fff;border:1px solid #E2E8F0;border-radius:14px;padding:18px;}
.panel-title{font-size:13px;font-weight:600;color:#374151;margin-bottom:14px;display:flex;align-items:center;gap:6px;}

/* 배지 */
.badge{display:inline-block;padding:3px 10px;border-radius:20px;font-size:11px;font-weight:600;letter-spacing:.3px;}
.b-blue{background:#DBEAFE;color:#1D4ED8;}
.b-green{background:#DCFCE7;color:#15803D;}
.b-amber{background:#FEF9C3;color:#A16207;}
.b-red{background:#FFE4E6;color:#BE123C;}
.b-gray{background:#F1F5F9;color:#475569;}
.b-teal{background:#CCFBF1;color:#0F766E;}

/* 과제 카드 */
.task-card{background:#fff;border:1px solid #E2E8F0;border-radius:12px;padding:14px 16px;margin-bottom:10px;border-left:4px solid #2563EB;}
.task-card.done{border-left-color:#15803D;opacity:.8;}
.task-card.overdue{border-left-color:#BE123C;}
.task-title{font-size:13px;font-weight:600;color:#1E293B;}
.task-meta{font-size:11px;color:#6B7280;margin-top:3px;}

/* 플래시카드 */
.flashcard{background:linear-gradient(135deg,#EFF6FF,#DBEAFE);border:1px solid #BFDBFE;border-radius:18px;padding:50px 36px;text-align:center;min-height:200px;display:flex;flex-direction:column;align-items:center;justify-content:center;margin:14px 0;}
.fc-word{font-size:2.2rem;font-weight:700;color:#1E3A8A;margin-bottom:8px;}
.fc-meaning{font-size:1.2rem;color:#2563EB;margin-top:12px;}
.fc-hint{font-size:0.8rem;color:#93C5FD;margin-top:18px;}

/* 퀴즈/시험 */
.exam-box{background:#F8FAFC;border:1px solid #E2E8F0;border-radius:12px;padding:20px;margin:14px 0;line-height:1.9;font-size:1.05rem;color:#334155;}
.progress-bg{background:#E2E8F0;border-radius:8px;height:7px;margin:6px 0;overflow:hidden;}
.progress-fill{background:linear-gradient(90deg,#2563EB,#06B6D4);height:100%;border-radius:8px;}
.result-ok{background:#DCFCE7;border:1px solid #BBF7D0;border-radius:10px;padding:12px;color:#15803D;font-weight:600;text-align:center;}
.result-ng{background:#FFE4E6;border:1px solid #FECDD3;border-radius:10px;padding:12px;color:#BE123C;font-weight:600;text-align:center;}

/* 공지 알림 */
.alert-task{background:#EFF6FF;border:1px solid #BFDBFE;border-radius:10px;padding:12px 16px;margin-bottom:8px;display:flex;align-items:center;justify-content:space-between;}
.alert-task.urgent{background:#FFF7ED;border-color:#FED7AA;}
.alert-task.overdue{background:#FFF1F2;border-color:#FECDD3;}

h1,h2,h3,h4,p,label,.stMarkdown{color:#374151!important;}
.stTextInput>div>div>input{background:#F8FAFC!important;color:#1E293B!important;border:1px solid #E2E8F0!important;border-radius:8px!important;}
.stButton>button{background:linear-gradient(135deg,#1D4ED8,#2563EB)!important;color:#fff!important;border:none!important;border-radius:8px!important;font-weight:600!important;}
.stButton>button:hover{opacity:.92!important;}
.stSelectbox>div>div{background:#F8FAFC!important;}
hr{border-color:#E2E8F0!important;}
</style>
""", unsafe_allow_html=True)

# ── 데이터 경로 ──────────────────────────────────────────────────────
DATA = "data"
os.makedirs(DATA, exist_ok=True)
USERS_F    = f"{DATA}/users.json"
VOCAB_F    = f"{DATA}/vocab.json"
RESULTS_F  = f"{DATA}/results.json"
PASSAGES_F = f"{DATA}/passages.json"
MOCK_F     = f"{DATA}/mock_passages.json"
VAR_F      = f"{DATA}/variation.json"
MVAR_F     = f"{DATA}/mock_variation.json"
TASKS_F    = f"{DATA}/tasks.json"

def load(p, d): return json.load(open(p, encoding="utf-8")) if os.path.exists(p) else d
def save(p, d): json.dump(d, open(p,"w",encoding="utf-8"), ensure_ascii=False, indent=2)
def hp(pw): return hashlib.sha256(pw.encode()).hexdigest()

# ── 초기 admin ───────────────────────────────────────────────────────
users = load(USERS_F, {})
if "admin" not in users:
    users["admin"] = {"password": hp("admin1234"), "role": "teacher", "name": "선생님"}
    save(USERS_F, users)

# ── 샘플 데이터 ──────────────────────────────────────────────────────
SAMPLE_VOCAB = [
    {"word":"ambiguous","meaning":"모호한","example":"The instructions were ambiguous.","category":"수능"},
    {"word":"contemporary","meaning":"현대의","example":"Contemporary art is fascinating.","category":"수능"},
    {"word":"inevitable","meaning":"불가피한","example":"Change is inevitable.","category":"수능"},
    {"word":"profound","meaning":"심오한","example":"It had a profound impact.","category":"내신"},
    {"word":"perseverance","meaning":"인내","example":"Success requires perseverance.","category":"내신"},
    {"word":"eloquent","meaning":"웅변적인","example":"She gave an eloquent speech.","category":"모의고사"},
    {"word":"meticulous","meaning":"꼼꼼한","example":"He is meticulous about details.","category":"모의고사"},
    {"word":"alleviate","meaning":"완화하다","example":"This will alleviate the pain.","category":"수능"},
]
SAMPLE_PASSAGES = [
    {"id":"p1","title":"2024 수능 18번 유형","category":"수능",
     "full_text":"The key to success is not talent but perseverance. Those who consistently put in effort over time will eventually surpass those who rely solely on natural ability. Meticulous planning and an eloquent presentation further distinguish great achievers.",
     "blanks":[{"word":"perseverance","hint":"인내"},{"word":"meticulous","hint":"꼼꼼한"},{"word":"eloquent","hint":"웅변적인"}]},
    {"id":"p2","title":"고2 내신 Unit 3","category":"내신",
     "full_text":"Climate change is an inevitable consequence of industrial development. Scientists have observed this profound phenomenon across the globe. However, ambiguous policies make it difficult to alleviate its effects.",
     "blanks":[{"word":"inevitable","hint":"불가피한"},{"word":"profound","hint":"심오한"},{"word":"alleviate","hint":"완화하다"}]},
]

# ── 세션 초기화 ───────────────────────────────────────────────────────
for k,v in {"logged_in":False,"username":"","role":"","user_name":"",
            "card_index":0,"show_answer":False,"shuffled_vocab":[],
            "quiz_index":0,"quiz_score":0,"quiz_answered":False,"quiz_result":None,"quiz_choices":[],
            "exam_passage":None,"exam_answers":{},"exam_submitted":False,"exam_score":None,
            "selected_category":"전체","api_key":"",
            "pdf_extracted":"","pdf_blanks_suggested":""}.items():
    if k not in st.session_state: st.session_state[k] = v

# ── 헬퍼 ─────────────────────────────────────────────────────────────
def get_vocab():
    data = load(VOCAB_F, SAMPLE_VOCAB)
    cat  = st.session_state.get("selected_category","전체")
    return [v for v in data if v.get("category")==cat] if cat!="전체" else data

def get_passages(): return load(PASSAGES_F, SAMPLE_PASSAGES)

def get_tasks():
    tasks = load(TASKS_F, [])
    today = date.today().isoformat()
    for t in tasks:
        if t.get("due") and t["due"] < today and t["status"] == "active":
            t["status"] = "overdue"
    return tasks

def pending_tasks(uid):
    tasks = get_tasks()
    results = load(RESULTS_F, {})
    my_done = {r["task_id"] for r in results.get(uid,[]) if r.get("task_id")}
    return [t for t in tasks if t["status"] in ("active","overdue") and t["id"] not in my_done]

# ── 로그인 ────────────────────────────────────────────────────────────
if not st.session_state.logged_in:
    st.markdown('<div style="text-align:center;padding:40px 0 10px;"><span style="font-size:2rem;font-weight:700;color:#1E3A8A;">📖 영어 암기 스튜디오</span></div>', unsafe_allow_html=True)
    st.markdown('<div style="text-align:center;color:#6B7280;font-size:0.85rem;margin-bottom:32px;">English Memory Studio · 고등학생 전용</div>', unsafe_allow_html=True)
    c1,c2,c3 = st.columns([1,1.1,1])
    with c2:
        with st.container():
            st.markdown('<div class="panel">', unsafe_allow_html=True)
            st.markdown("#### 🔐 로그인")
            uid = st.text_input("아이디", placeholder="student01 또는 admin")
            pw  = st.text_input("비밀번호", type="password")
            if st.button("로그인", use_container_width=True):
                users = load(USERS_F, {})
                if uid in users and users[uid]["password"] == hp(pw):
                    st.session_state.update(logged_in=True, username=uid,
                        role=users[uid]["role"], user_name=users[uid]["name"])
                    st.rerun()
                else: st.error("아이디 또는 비밀번호가 틀렸습니다.")
            st.markdown('</div>', unsafe_allow_html=True)
        st.markdown('<p style="color:#9CA3AF;font-size:0.78rem;text-align:center;margin-top:10px;">기본 관리자: admin / admin1234</p>', unsafe_allow_html=True)
    st.stop()

# ── 사이드바 ──────────────────────────────────────────────────────────
with st.sidebar:
    st.markdown(f'''
    <div class="sb-logo">
      <div class="sb-logo-icon"><span style="color:#fff;font-size:18px;">📖</span></div>
      <div class="sb-app-name">영어 암기 스튜디오</div>
      <div class="sb-app-sub">English Memory Studio</div>
    </div>
    <div class="sb-user">
      <div class="sb-avatar">{st.session_state.user_name[:2]}</div>
      <div><div class="sb-name">{st.session_state.user_name}</div>
      <div class="sb-role">{"선생님" if st.session_state.role=="teacher" else "학생"}</div></div>
    </div>
    ''', unsafe_allow_html=True)

    is_teacher = st.session_state.role == "teacher"
    st.markdown('<div class="sb-section">학습</div>', unsafe_allow_html=True)
    study_menus = ["🏠 홈","🃏 플래시카드","✏️ 빈칸 퀴즈","📝 본문 암기 시험",
                   "📄 모의고사 지문","📖 본문 변형문제","🧪 모의고사 변형문제"]
    if is_teacher:
        st.markdown('<div class="sb-section">관리</div>', unsafe_allow_html=True)
        admin_menus = ["📤 자료 업로드","📋 과제 관리","👥 학생 관리","📊 성적 관리"]
        menu = st.radio("메뉴", study_menus + admin_menus, label_visibility="collapsed")
    else:
        extra = ["📋 내 과제"]
        menu  = st.radio("메뉴", study_menus + extra, label_visibility="collapsed")

    st.markdown("---")
    cats = list(set(v.get("category","") for v in load(VOCAB_F, SAMPLE_VOCAB)))
    sc   = st.selectbox("📌 범위", ["전체"]+cats)
    st.session_state.selected_category = sc

    # 미완료 과제 알림 (학생)
    if not is_teacher:
        pending = pending_tasks(st.session_state.username)
        if pending:
            st.markdown(f'<div style="background:#FFF7ED;border:1px solid #FED7AA;border-radius:8px;padding:8px 12px;margin:8px 0;"><span style="color:#9A3412;font-size:12px;font-weight:600;">⚠️ 미완료 과제 {len(pending)}개</span></div>', unsafe_allow_html=True)

    st.markdown("---")
    if st.button("🚪 로그아웃", use_container_width=True):
        for k in list(st.session_state.keys()): del st.session_state[k]
        st.rerun()

# ══════════════════════════════════════════════════════════════════════
# 홈 대시보드
# ══════════════════════════════════════════════════════════════════════
if menu == "🏠 홈":
    vocab     = get_vocab()
    passages  = get_passages()
    results   = load(RESULTS_F, {})
    uid       = st.session_state.username
    my_res    = results.get(uid, [])
    avg       = int(sum(r["score"] for r in my_res)/len(my_res)) if my_res else 0
    today_str = date.today().strftime("%Y년 %m월 %d일")

    if is_teacher:
        users_all    = load(USERS_F, {})
        students     = {k:v for k,v in users_all.items() if v["role"]=="student"}
        tasks        = get_tasks()
        active_tasks = [t for t in tasks if t["status"]=="active"]

        # 과제별 완료율 계산
        task_rows = ""
        for t in active_tasks[:3]:
            done_cnt = sum(1 for suid in students if any(r.get("task_id")==t["id"] for r in results.get(suid,[])))
            total_s  = max(len(students), 1)
            pct      = int(done_cnt/total_s*100)
            bar_color = "#15803D" if pct>=80 else "#2563EB" if pct>=40 else "#DC2626"
            task_rows += f"""
            <div style="margin-bottom:14px;">
              <div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:4px;">
                <span style="font-size:13px;font-weight:600;color:#1E293B;">{t['title']}</span>
                <span style="font-size:12px;color:#6B7280;">마감 {t['due']}</span>
              </div>
              <div style="background:#E2E8F0;border-radius:6px;height:8px;overflow:hidden;">
                <div style="width:{pct}%;height:100%;background:{bar_color};border-radius:6px;"></div>
              </div>
              <div style="font-size:11px;color:#6B7280;margin-top:3px;">{done_cnt}/{total_s}명 완료 ({pct}%)</div>
            </div>"""

        # 학생별 평균 top3
        student_rows = ""
        rank_data = []
        for suid, sinfo in students.items():
            s_res = results.get(suid, [])
            if s_res:
                s_avg = int(sum(r["score"] for r in s_res)/len(s_res))
                rank_data.append((sinfo["name"], s_avg, len(s_res)))
        rank_data.sort(key=lambda x: -x[1])
        medals = ["🥇","🥈","🥉"]
        for i, (nm, sc, cnt) in enumerate(rank_data[:5]):
            color = "#15803D" if sc>=80 else "#A16207" if sc>=60 else "#DC2626"
            medal = medals[i] if i<3 else f"{i+1}위"
            student_rows += f"""
            <div style="display:flex;align-items:center;gap:10px;padding:8px 0;border-bottom:1px solid #F1F5F9;">
              <span style="font-size:16px;min-width:28px;">{medal}</span>
              <span style="flex:1;font-size:13px;font-weight:600;color:#1E293B;">{nm}</span>
              <span style="font-size:13px;font-weight:700;color:{color};">{sc}점</span>
              <span style="font-size:11px;color:#9CA3AF;">{cnt}회</span>
            </div>"""
        if not student_rows:
            student_rows = '<p style="color:#9CA3AF;font-size:13px;text-align:center;padding:20px 0;">아직 시험 기록이 없어요</p>'

        st.markdown(f"""
        <style>
        .dash-grid{{display:grid;grid-template-columns:repeat(4,1fr);gap:14px;margin-bottom:20px;}}
        .dash-card{{background:#fff;border:1px solid #E2E8F0;border-radius:14px;padding:18px 20px;}}
        .dash-icon{{width:40px;height:40px;border-radius:11px;display:flex;align-items:center;justify-content:center;font-size:19px;margin-bottom:12px;}}
        .dash-val{{font-size:2rem;font-weight:700;color:#1E293B;line-height:1;}}
        .dash-label{{font-size:12px;color:#6B7280;margin-top:5px;}}
        .dash-panel{{background:#fff;border:1px solid #E2E8F0;border-radius:14px;padding:20px;}}
        .dash-panel-title{{font-size:14px;font-weight:700;color:#1E293B;margin-bottom:16px;display:flex;align-items:center;gap:6px;}}
        .dash-two{{display:grid;grid-template-columns:1fr 1fr;gap:14px;}}
        </style>

        <div style="margin-bottom:6px;">
          <span style="font-size:1.4rem;font-weight:700;color:#1E3A5F;">안녕하세요, {st.session_state.user_name} 선생님 👋</span>
        </div>
        <div style="font-size:0.82rem;color:#6B7280;margin-bottom:22px;">{today_str}</div>

        <div class="dash-grid">
          <div class="dash-card">
            <div class="dash-icon" style="background:#DBEAFE;">📚</div>
            <div class="dash-val">{len(vocab)}</div>
            <div class="dash-label">등록 단어</div>
          </div>
          <div class="dash-card">
            <div class="dash-icon" style="background:#DCFCE7;">📄</div>
            <div class="dash-val">{len(passages)}</div>
            <div class="dash-label">등록 지문</div>
          </div>
          <div class="dash-card">
            <div class="dash-icon" style="background:#FEF9C3;">👥</div>
            <div class="dash-val">{len(students)}</div>
            <div class="dash-label">학생 수</div>
          </div>
          <div class="dash-card">
            <div class="dash-icon" style="background:#CCFBF1;">📋</div>
            <div class="dash-val">{len(active_tasks)}</div>
            <div class="dash-label">진행 과제</div>
          </div>
        </div>

        <div class="dash-two">
          <div class="dash-panel">
            <div class="dash-panel-title">📋 과제 진행 현황</div>
            {task_rows if task_rows else '<p style="color:#9CA3AF;font-size:13px;text-align:center;padding:20px 0;">진행 중인 과제 없음</p>'}
          </div>
          <div class="dash-panel">
            <div class="dash-panel-title">🏆 학생 성적 랭킹</div>
            {student_rows}
          </div>
        </div>
        """, unsafe_allow_html=True)

    else:
        # 학생 대시보드
        pending = pending_tasks(uid)

        # 최근 5개 점수 스파크라인
        recent_scores = [r["score"] for r in my_res[-5:]]
        score_bars = ""
        for sc in recent_scores:
            h = max(int(sc * 0.5), 4)
            color = "#15803D" if sc>=80 else "#2563EB" if sc>=60 else "#DC2626"
            score_bars += f'<div style="width:28px;height:{h}px;background:{color};border-radius:4px 4px 0 0;" title="{sc}점"></div>'

        # 미완료 과제 카드
        pending_html = ""
        for t in pending[:3]:
            due = t.get("due","")
            is_over = due < date.today().isoformat()
            badge_color = "#DC2626" if is_over else "#2563EB"
            badge_bg    = "#FFE4E6" if is_over else "#DBEAFE"
            badge_text  = f"마감초과 ({due})" if is_over else f"마감 {due}"
            pending_html += f"""
            <div style="display:flex;align-items:center;justify-content:space-between;
                        padding:12px 14px;background:#F8FAFC;border-radius:10px;
                        border-left:3px solid {badge_color};margin-bottom:8px;">
              <div>
                <div style="font-size:13px;font-weight:600;color:#1E293B;">{t["title"]}</div>
                <div style="font-size:11px;color:#6B7280;margin-top:2px;">{t["type_label"]}</div>
              </div>
              <span style="font-size:11px;font-weight:600;color:{badge_color};
                           background:{badge_bg};padding:3px 10px;border-radius:20px;">{badge_text}</span>
            </div>"""
        if not pending_html:
            pending_html = '<div style="text-align:center;padding:24px 0;color:#15803D;font-weight:600;">🎉 모든 과제 완료!</div>'

        # 최근 시험 기록
        recent_html = ""
        for r in reversed(my_res[-4:]):
            sc = r["score"]
            color = "#15803D" if sc>=80 else "#A16207" if sc>=60 else "#DC2626"
            bg    = "#DCFCE7" if sc>=80 else "#FEF9C3" if sc>=60 else "#FFE4E6"
            recent_html += f"""
            <div style="display:flex;align-items:center;justify-content:space-between;
                        padding:9px 0;border-bottom:1px solid #F1F5F9;">
              <div>
                <div style="font-size:13px;font-weight:500;color:#1E293B;">{r["passage_title"]}</div>
                <div style="font-size:11px;color:#9CA3AF;">{r["date"]}</div>
              </div>
              <span style="font-size:13px;font-weight:700;color:{color};
                           background:{bg};padding:4px 12px;border-radius:20px;">{sc}점</span>
            </div>"""
        if not recent_html:
            recent_html = '<p style="color:#9CA3AF;font-size:13px;text-align:center;padding:20px 0;">아직 기록 없음</p>'

        st.markdown(f"""
        <style>
        .dash-grid{{display:grid;grid-template-columns:repeat(4,1fr);gap:14px;margin-bottom:20px;}}
        .dash-card{{background:#fff;border:1px solid #E2E8F0;border-radius:14px;padding:18px 20px;}}
        .dash-icon{{width:40px;height:40px;border-radius:11px;display:flex;align-items:center;justify-content:center;font-size:19px;margin-bottom:12px;}}
        .dash-val{{font-size:2rem;font-weight:700;color:#1E293B;line-height:1;}}
        .dash-label{{font-size:12px;color:#6B7280;margin-top:5px;}}
        .dash-panel{{background:#fff;border:1px solid #E2E8F0;border-radius:14px;padding:20px;}}
        .dash-panel-title{{font-size:14px;font-weight:700;color:#1E293B;margin-bottom:16px;}}
        .dash-two{{display:grid;grid-template-columns:1fr 1fr;gap:14px;}}
        </style>

        <div style="margin-bottom:6px;">
          <span style="font-size:1.4rem;font-weight:700;color:#1E3A5F;">안녕하세요, {st.session_state.user_name}님 👋</span>
        </div>
        <div style="font-size:0.82rem;color:#6B7280;margin-bottom:22px;">{today_str}</div>

        <div class="dash-grid">
          <div class="dash-card">
            <div class="dash-icon" style="background:#DBEAFE;">📚</div>
            <div class="dash-val">{len(vocab)}</div>
            <div class="dash-label">학습 단어</div>
          </div>
          <div class="dash-card">
            <div class="dash-icon" style="background:#DCFCE7;">📝</div>
            <div class="dash-val">{len(my_res)}</div>
            <div class="dash-label">응시 횟수</div>
          </div>
          <div class="dash-card">
            <div class="dash-icon" style="background:#FEF9C3;">📊</div>
            <div class="dash-val">{avg}%</div>
            <div class="dash-label">평균 점수</div>
            <div style="display:flex;align-items:flex-end;gap:3px;margin-top:8px;height:28px;">
              {score_bars}
            </div>
          </div>
          <div class="dash-card">
            <div class="dash-icon" style="background:#FFE4E6;">⚠️</div>
            <div class="dash-val">{len(pending)}</div>
            <div class="dash-label">미완료 과제</div>
          </div>
        </div>

        <div class="dash-two">
          <div class="dash-panel">
            <div class="dash-panel-title">⚠️ 미완료 과제</div>
            {pending_html}
          </div>
          <div class="dash-panel">
            <div class="dash-panel-title">📋 최근 시험 기록</div>
            {recent_html}
          </div>
        </div>
        """, unsafe_allow_html=True)

# ══════════════════════════════════════════════════════════════════════
# 자료 업로드 (선생님)
# ══════════════════════════════════════════════════════════════════════
elif menu == "📤 자료 업로드" and is_teacher:
    st.markdown('<div class="page-title">📤 자료 업로드</div>', unsafe_allow_html=True)
    tab1, tab2 = st.tabs(["📚 단어 업로드", "📄 지문 업로드"])

    with tab1:
        st.markdown('<div class="panel">', unsafe_allow_html=True)
        st.code("word,meaning,example,category\nambiguous,모호한,The text was ambiguous.,수능", language="text")
        up = st.file_uploader("CSV/Excel", type=["csv","xlsx"], key="vocab_up")
        if up:
            try:
                df = pd.read_csv(up) if up.name.endswith(".csv") else pd.read_excel(up)
                if "category" not in df.columns: df["category"]="기본"
                if "example"  not in df.columns: df["example"]=""
                save(VOCAB_F, df.to_dict("records"))
                st.success(f"✅ {len(df)}개 저장 완료!")
                st.dataframe(df.head(8), use_container_width=True, hide_index=True)
            except Exception as e: st.error(str(e))
        st.markdown('</div>', unsafe_allow_html=True)

    with tab2:
        # API 키
        st.markdown('<div class="panel" style="margin-bottom:14px;">', unsafe_allow_html=True)
        st.markdown("#### 🔑 Claude API 키")
        ak = st.text_input("API 키", value=st.session_state.api_key, type="password", placeholder="sk-ant-...")
        c1,c2 = st.columns([1,3])
        with c1:
            if st.button("저장"):
                st.session_state.api_key = ak; st.success("저장됨")
        with c2:
            if st.session_state.api_key:
                st.markdown('<span class="badge b-green">🟢 연결됨</span>', unsafe_allow_html=True)
            else:
                st.markdown('<span class="badge b-red">🔴 없음 — 수동 등록만 가능</span>', unsafe_allow_html=True)
        st.markdown('</div>', unsafe_allow_html=True)

        # PDF 업로드
        st.markdown('<div class="panel" style="margin-bottom:14px;">', unsafe_allow_html=True)
        st.markdown("#### 📄 PDF 업로드")
        pdf_f = st.file_uploader("PDF", type=["pdf"], key="passage_pdf")
        if pdf_f:
            if not PDF_OK: st.error("pypdf 미설치")
            else:
                reader = PdfReader(pdf_f)
                ext = " ".join(p.extract_text() or "" for p in reader.pages).strip()
                if ext:
                    st.success(f"✅ {len(ext)}자 추출")
                    st.session_state.pdf_extracted = st.text_area("추출 본문 (수정 가능)", value=ext, height=180)
                    if st.session_state.api_key:
                        if st.button("🤖 Claude 빈칸 자동 추천"):
                            with st.spinner("분석 중..."):
                                try:
                                    import anthropic
                                    client = anthropic.Anthropic(api_key=st.session_state.api_key)
                                    msg = client.messages.create(
                                        model="claude-haiku-4-5-20251001", max_tokens=500,
                                        messages=[{"role":"user","content":f"다음 영어 지문에서 수능/내신 핵심 단어 5~8개를 골라 JSON 배열로만 답하세요: [{{\"word\":\"단어\",\"hint\":\"한국어뜻\"}}]\n\n{ext[:2000]}"}])
                                    raw = msg.content[0].text.strip()
                                    m   = re.search(r'\[.*\]', raw, re.DOTALL)
                                    if m:
                                        suggested = json.loads(m.group())
                                        st.session_state.pdf_blanks_suggested = "\n".join(f"{b['word']},{b['hint']}" for b in suggested)
                                        st.success(f"✅ {len(suggested)}개 추천!")
                                except Exception as e: st.error(str(e))
                    else: st.info("💡 API 키 입력 시 빈칸 자동 추천 가능")
                else: st.error("텍스트 추출 불가 (스캔 PDF는 미지원)")
        st.markdown('</div>', unsafe_allow_html=True)

        # 지문 저장 폼
        st.markdown('<div class="panel">', unsafe_allow_html=True)
        st.markdown("#### ✏️ 지문 등록")
        with st.form("passage_form"):
            c1,c2 = st.columns(2)
            with c1: p_id    = st.text_input("지문 ID (영문)", placeholder="p3")
            with c2: p_title = st.text_input("제목", placeholder="2024 수능 29번")
            p_cat  = st.selectbox("범위", ["수능","내신","모의고사"])
            p_text = st.text_area("본문", value=st.session_state.pdf_extracted, height=140)
            p_braw = st.text_area("빈칸 단어 (word,힌트 한 줄씩)", value=st.session_state.pdf_blanks_suggested, height=90,
                                   placeholder="perseverance,인내\nmeticulous,꼼꼼한")
            if st.form_submit_button("💾 저장", use_container_width=True) and p_id and p_text and p_braw:
                blanks = [{"word":l.split(",",1)[0].strip(),"hint":l.split(",",1)[1].strip() if "," in l else ""}
                          for l in p_braw.strip().splitlines()]
                ps = [p for p in get_passages() if p["id"]!=p_id]
                ps.append({"id":p_id,"title":p_title,"category":p_cat,"full_text":p_text,"blanks":blanks})
                save(PASSAGES_F, ps)
                st.session_state.pdf_extracted = ""; st.session_state.pdf_blanks_suggested = ""
                st.success(f"✅ '{p_title}' 저장!")
        st.markdown('</div>', unsafe_allow_html=True)

# ══════════════════════════════════════════════════════════════════════
# 과제 관리 (선생님)
# ══════════════════════════════════════════════════════════════════════
elif menu == "📋 과제 관리" and is_teacher:
    st.markdown('<div class="page-title">📋 과제 관리</div>', unsafe_allow_html=True)
    st.markdown('<div class="page-sub">반 전체에게 숙제/시험을 일괄 부과하고 결과를 확인하세요</div>', unsafe_allow_html=True)

    tasks   = get_tasks()
    results = load(RESULTS_F, {})
    users_all = load(USERS_F, {})
    students  = {k:v for k,v in users_all.items() if v["role"]=="student"}

    tab1, tab2 = st.tabs(["➕ 과제 생성", "📊 과제 현황"])

    with tab1:
        st.markdown('<div class="panel">', unsafe_allow_html=True)
        with st.form("task_form"):
            t_title = st.text_input("과제 제목", placeholder="6월 2주차 본문 암기 시험")
            c1,c2   = st.columns(2)
            with c1: t_type = st.selectbox("유형", ["본문 암기 시험","빈칸 퀴즈","플래시카드 학습"])
            with c2: t_due  = st.date_input("마감일", value=date.today())

            # 유형별 설정
            passages = get_passages()
            if t_type in ("본문 암기 시험","빈칸 퀴즈"):
                p_titles = [p["title"] for p in passages]
                t_target = st.selectbox("대상 지문", p_titles) if p_titles else st.text_input("지문 없음 — 먼저 등록하세요")
            else:
                cats_list = list(set(v.get("category","") for v in load(VOCAB_F, SAMPLE_VOCAB)))
                t_target  = st.selectbox("단어 범위", ["전체"]+cats_list)

            t_desc = st.text_area("학생에게 표시할 안내 메시지", height=70, placeholder="이번 주 수능 본문 3지문 암기 시험입니다.")

            if st.form_submit_button("📣 전체 학생에게 부과", use_container_width=True):
                if t_title:
                    new_task = {
                        "id":       f"task_{len(tasks)+1}_{int(datetime.now().timestamp())}",
                        "title":    t_title,
                        "type":     t_type,
                        "type_label": t_type,
                        "target":   t_target,
                        "due":      t_due.isoformat(),
                        "desc":     t_desc,
                        "status":   "active",
                        "created":  datetime.now().strftime("%Y-%m-%d %H:%M"),
                    }
                    tasks.append(new_task)
                    save(TASKS_F, tasks)
                    st.success(f"✅ '{t_title}' — {len(students)}명에게 부과 완료!")
                    st.rerun()
        st.markdown('</div>', unsafe_allow_html=True)

    with tab2:
        if not tasks:
            st.info("부과된 과제가 없습니다.")
        else:
            for t in reversed(tasks):
                today = date.today().isoformat()
                status_badge = '<span class="badge b-green">진행중</span>' if t["status"]=="active" else \
                               '<span class="badge b-red">마감 초과</span>' if t["status"]=="overdue" else \
                               '<span class="badge b-gray">종료</span>'
                due_badge = f'<span class="badge b-amber">마감 {t["due"]}</span>'

                # 완료 학생 수
                done_students = [uid for uid,rlist in results.items()
                                 if uid in students and any(r.get("task_id")==t["id"] for r in rlist)]
                completion    = f"{len(done_students)}/{len(students)}명 완료"

                with st.expander(f"{t['title']}  —  {completion}"):
                    c1,c2,c3 = st.columns(3)
                    c1.markdown(status_badge, unsafe_allow_html=True)
                    c2.markdown(due_badge, unsafe_allow_html=True)
                    c3.markdown(f'<span class="badge b-blue">{t["type_label"]}</span>', unsafe_allow_html=True)
                    st.markdown(f"**대상:** {t['target']}  |  **생성:** {t['created']}")
                    if t.get("desc"): st.markdown(f"📌 {t['desc']}")
                    st.markdown("---")

                    # 학생별 완료 현황
                    rows = []
                    for suid, sinfo in students.items():
                        s_res = [r for r in results.get(suid,[]) if r.get("task_id")==t["id"]]
                        if s_res:
                            best = max(r["score"] for r in s_res)
                            rows.append({"이름":sinfo["name"],"상태":"✅ 완료",f"최고점수":f"{best}%","응시횟수":len(s_res)})
                        else:
                            rows.append({"이름":sinfo["name"],"상태":"❌ 미완료","최고점수":"-","응시횟수":0})
                    if rows:
                        st.dataframe(pd.DataFrame(rows), use_container_width=True, hide_index=True)

                    # 종료 처리
                    if t["status"] != "closed":
                        if st.button("🔒 과제 종료", key=f"close_{t['id']}"):
                            for tk in tasks:
                                if tk["id"] == t["id"]: tk["status"] = "closed"
                            save(TASKS_F, tasks); st.rerun()

# ══════════════════════════════════════════════════════════════════════
# 내 과제 (학생)
# ══════════════════════════════════════════════════════════════════════
elif menu == "📋 내 과제":
    st.markdown('<div class="page-title">📋 내 과제</div>', unsafe_allow_html=True)
    uid     = st.session_state.username
    tasks   = get_tasks()
    results = load(RESULTS_F, {})
    my_done = {r["task_id"] for r in results.get(uid,[]) if r.get("task_id")}
    today   = date.today().isoformat()

    active   = [t for t in tasks if t["id"] not in my_done and t["status"] in ("active","overdue")]
    done_list= [t for t in tasks if t["id"] in my_done]
    closed   = [t for t in tasks if t["status"]=="closed" and t["id"] not in my_done]

    tab1, tab2 = st.tabs([f"🔔 미완료 ({len(active)+len(closed)})", f"✅ 완료 ({len(done_list)})"])

    with tab1:
        if not active and not closed:
            st.success("🎉 모든 과제를 완료했어요!")
        for t in active:
            is_over = t["due"] < today
            border  = "#BE123C" if is_over else "#2563EB"
            badge   = f'<span class="badge b-red">마감 초과 ({t["due"]})</span>' if is_over else f'<span class="badge b-amber">마감 {t["due"]}</span>'
            st.markdown(f'<div class="task-card {"overdue" if is_over else ""}"><div style="display:flex;justify-content:space-between;align-items:start;"><div><div class="task-title">{t["title"]}</div><div class="task-meta">{t["type_label"]} · {t.get("desc","")}</div></div>{badge}</div></div>', unsafe_allow_html=True)

            # 바로 응시 버튼
            passages = get_passages()
            match_p  = next((p for p in passages if p["title"]==t.get("target","")), None)
            if t["type"] == "본문 암기 시험" and match_p:
                if st.button(f"✏️ 지금 응시하기", key=f"take_{t['id']}"):
                    st.session_state.exam_passage   = match_p
                    st.session_state.exam_answers   = {}
                    st.session_state.exam_submitted = False
                    st.session_state.exam_score     = None
                    st.session_state.current_task_id= t["id"]
                    st.rerun()
            elif t["type"] == "빈칸 퀴즈":
                st.info(f"빈칸 퀴즈 탭에서 '{t.get('target','')}' 범위를 공부하세요.")

        for t in closed:
            st.markdown(f'<div class="task-card overdue"><div class="task-title">{t["title"]}</div><div class="task-meta">⛔ 종료된 과제 · 미응시</div></div>', unsafe_allow_html=True)

    with tab2:
        if not done_list:
            st.info("완료한 과제가 없습니다.")
        for t in done_list:
            t_res = [r for r in results.get(uid,[]) if r.get("task_id")==t["id"]]
            best  = max(r["score"] for r in t_res) if t_res else 0
            st.markdown(f'<div class="task-card done"><div style="display:flex;justify-content:space-between;"><div><div class="task-title">{t["title"]}</div><div class="task-meta">{t["type_label"]} · {t["due"]} 마감</div></div><span class="badge b-green">{best}점</span></div></div>', unsafe_allow_html=True)

# ══════════════════════════════════════════════════════════════════════
# 플래시카드
# ══════════════════════════════════════════════════════════════════════
# ══════════════════════════════════════════════════════════════════════
# 본문 암기 시험 공통 함수
# ══════════════════════════════════════════════════════════════════════
def run_exam(task_id=None):
    passages = get_passages()
    uid      = st.session_state.username

    if not st.session_state.exam_passage:
        st.markdown("#### 시험 볼 지문을 선택하세요")
        for p in passages:
            cat_b = "b-blue" if p["category"]=="수능" else "b-green" if p["category"]=="내신" else "b-amber"
            c1,c2 = st.columns([4,1])
            with c1:
                st.markdown(f'<div class="panel" style="margin-bottom:8px;border-left:3px solid #2563EB;"><span class="badge {cat_b}">{p["category"]}</span> &nbsp;<strong>{p["title"]}</strong><br><span style="color:#6B7280;font-size:11px;">빈칸 {len(p["blanks"])}개</span></div>', unsafe_allow_html=True)
            with c2:
                st.markdown("<br>", unsafe_allow_html=True)
                if st.button("시작 ▶", key=f"s_{p['id']}"):
                    st.session_state.exam_passage=p; st.session_state.exam_answers={}
                    st.session_state.exam_submitted=False; st.session_state.exam_score=None
                    if task_id: st.session_state.current_task_id=task_id
                    st.rerun()
    else:
        p = st.session_state.exam_passage
        st.markdown(f"#### 📄 {p['title']}")
        if not st.session_state.exam_submitted:
            disp = p["full_text"]
            for i,b in enumerate(p["blanks"]): disp=disp.replace(b["word"],f"**( {i+1} )**",1)
            st.markdown(f'<div class="exam-box">{disp}</div>', unsafe_allow_html=True)
            st.markdown("---")
            with st.form("exam_f"):
                answers={}
                cols=st.columns(2)
                for i,b in enumerate(p["blanks"]):
                    with cols[i%2]:
                        a=st.text_input(f"( {i+1} ) 힌트: {b['hint']}", key=f"ea_{i}", placeholder="영어 단어")
                        answers[str(i)]=a.strip().lower()
                if st.form_submit_button("✅ 제출", use_container_width=True):
                    correct=sum(1 for i,b in enumerate(p["blanks"]) if answers.get(str(i),"")==b["word"].lower())
                    total=len(p["blanks"]); score=int(correct/total*100)
                    results=load(RESULTS_F,{})
                    if uid not in results: results[uid]=[]
                    entry={"date":datetime.now().strftime("%Y-%m-%d %H:%M"),"passage_id":p["id"],
                           "passage_title":p["title"],"score":score,"correct":correct,"total":total,"answers":answers}
                    if st.session_state.get("current_task_id"):
                        entry["task_id"]=st.session_state.current_task_id
                    results[uid].append(entry)
                    save(RESULTS_F,results)
                    st.session_state.exam_submitted=True
                    st.session_state.exam_score={"score":score,"correct":correct,"total":total,"answers":answers}
                    st.session_state.current_task_id=None
                    st.rerun()
        else:
            sc=st.session_state.exam_score; pct=sc["score"]
            emoji="🔥" if pct==100 else "💪" if pct>=70 else "📚"
            st.markdown(f'<div class="flashcard"><div class="fc-word">{emoji} {sc["correct"]}/{sc["total"]}</div><div class="fc-meaning">점수: {pct}점</div></div>', unsafe_allow_html=True)
            st.markdown("#### 채점 결과")
            for i,b in enumerate(p["blanks"]):
                ok=sc["answers"].get(str(i),"")==b["word"].lower()
                color="#15803D" if ok else "#BE123C"
                icon="✅" if ok else "❌"
                st.markdown(f'<p style="color:{color};">{icon} ({i+1}) 정답: <strong>{b["word"]}</strong> / 내 답: {sc["answers"].get(str(i),"(미입력)")}</p>', unsafe_allow_html=True)
            c1,c2=st.columns(2)
            with c1:
                if st.button("🔄 다시 풀기"): st.session_state.exam_submitted=False; st.session_state.exam_answers={}; st.session_state.exam_score=None; st.rerun()
            with c2:
                if st.button("📋 목록으로"): st.session_state.exam_passage=None; st.rerun()

# ── 메뉴 라우팅 ──────────────────────────────────────────────────────
if menu == "🃏 플래시카드":
    vocab = get_vocab()
    st.markdown('<div class="page-title">🃏 플래시카드</div>', unsafe_allow_html=True)
    if not vocab:
        st.warning("단어 없음")
    else:
        if not st.session_state.shuffled_vocab or len(st.session_state.shuffled_vocab)!=len(vocab):
            st.session_state.shuffled_vocab = vocab.copy(); random.shuffle(st.session_state.shuffled_vocab)
        idx  = st.session_state.card_index % len(st.session_state.shuffled_vocab)
        card = st.session_state.shuffled_vocab[idx]
        pct  = (idx+1)/len(vocab)
        st.markdown(f'<div class="progress-bg"><div class="progress-fill" style="width:{pct*100:.1f}%"></div></div>', unsafe_allow_html=True)
        st.markdown(f'<p style="color:#9CA3AF;font-size:0.8rem;text-align:right;">{idx+1}/{len(vocab)}</p>', unsafe_allow_html=True)
        cat_b = f'<span class="badge b-blue">{card.get("category","")}</span>'
        if not st.session_state.show_answer:
            st.markdown(f'<div class="flashcard">{cat_b}<div class="fc-word">{card["word"]}</div><div class="fc-hint">▼ 뜻 보기를 눌러 확인</div></div>', unsafe_allow_html=True)
        else:
            st.markdown(f'<div class="flashcard">{cat_b}<div class="fc-word">{card["word"]}</div><div class="fc-meaning">🇰🇷 {card["meaning"]}</div><div class="fc-hint">{card.get("example","")}</div></div>', unsafe_allow_html=True)
        c1,c2,c3,c4 = st.columns(4)
        with c1:
            if st.button("◀ 이전"): st.session_state.card_index=max(0,st.session_state.card_index-1); st.session_state.show_answer=False; st.rerun()
        with c2:
            if st.button("🙈 숨기기" if st.session_state.show_answer else "👁 뜻 보기"): st.session_state.show_answer=not st.session_state.show_answer; st.rerun()
        with c3:
            if st.button("다음 ▶"): st.session_state.card_index+=1; st.session_state.show_answer=False; st.rerun()
        with c4:
            if st.button("🔀 셔플"): random.shuffle(st.session_state.shuffled_vocab); st.session_state.card_index=0; st.session_state.show_answer=False; st.rerun()

# ══════════════════════════════════════════════════════════════════════
# 빈칸 퀴즈
# ══════════════════════════════════════════════════════════════════════
elif menu == "✏️ 빈칸 퀴즈":
    vocab = get_vocab()
    st.markdown('<div class="page-title">✏️ 빈칸 퀴즈</div>', unsafe_allow_html=True)
    if len(vocab) < 4:
        st.warning("최소 4개 이상 단어 필요")
    else:
        if not st.session_state.quiz_choices or st.session_state.quiz_index>=len(vocab):
            st.session_state.quiz_index=0; st.session_state.quiz_score=0
            sv=vocab.copy(); random.shuffle(sv); st.session_state.shuffled_vocab=sv
            st.session_state.quiz_answered=False; st.session_state.quiz_result=None

        if st.session_state.quiz_index >= len(vocab):
            sc=st.session_state.quiz_score; tot=len(vocab); pct=int(sc/tot*100)
            st.markdown(f'<div class="flashcard"><div class="fc-word">🎯 {sc}/{tot}</div><div class="fc-meaning">정답률 {pct}%</div><div class="fc-hint">{"완벽해요! 🔥" if pct==100 else "잘 했어요! 💪" if pct>=70 else "다시 도전! 📚"}</div></div>', unsafe_allow_html=True)
            if st.button("🔄 다시 시작"): st.session_state.quiz_index=0; st.session_state.quiz_score=0; st.session_state.quiz_answered=False; st.session_state.quiz_result=None; st.session_state.quiz_choices=[]; st.rerun()
        else:
            idx=st.session_state.quiz_index; card=st.session_state.shuffled_vocab[idx]
            st.markdown(f'<div class="progress-bg"><div class="progress-fill" style="width:{idx/len(vocab)*100:.1f}%"></div></div>', unsafe_allow_html=True)
            st.markdown(f'<p style="color:#9CA3AF;font-size:0.8rem;text-align:right;">{idx+1}/{len(vocab)}</p>', unsafe_allow_html=True)
            if not st.session_state.quiz_choices:
                wrong=[v["word"] for v in vocab if v["word"]!=card["word"]]
                choices=random.sample(wrong,min(3,len(wrong)))+[card["word"]]; random.shuffle(choices)
                st.session_state.quiz_choices=choices
            q_text=card.get("example",f"'{card['meaning']}'을 뜻하는 단어는?").replace(card["word"],"______")
            st.markdown(f'<div class="exam-box"><span class="badge b-blue">{card.get("category","")}</span><br><br>{q_text}</div>', unsafe_allow_html=True)
            st.markdown(f"**힌트:** {card['meaning']}")
            if not st.session_state.quiz_answered:
                c1,c2=st.columns(2)
                for i,ch in enumerate(st.session_state.quiz_choices):
                    with [c1,c2][i%2]:
                        if st.button(f"  {ch}  ", key=f"c_{i}"):
                            st.session_state.quiz_answered=True
                            st.session_state.quiz_result="correct" if ch==card["word"] else "wrong"
                            if ch==card["word"]: st.session_state.quiz_score+=1
                            st.rerun()
            else:
                if st.session_state.quiz_result=="correct":
                    st.markdown('<div class="result-ok">✅ 정답!</div>', unsafe_allow_html=True)
                else:
                    st.markdown(f'<div class="result-ng">❌ 오답 — 정답: <strong>{card["word"]}</strong></div>', unsafe_allow_html=True)
                if st.button("다음 ▶"): st.session_state.quiz_index+=1; st.session_state.quiz_answered=False; st.session_state.quiz_result=None; st.session_state.quiz_choices=[]; st.rerun()

elif menu == "📝 본문 암기 시험":
    st.markdown('<div class="page-title">📝 본문 암기 시험</div>', unsafe_allow_html=True)
    run_exam(task_id=st.session_state.get("current_task_id"))

# ══════════════════════════════════════════════════════════════════════
# 모의고사 지문
# ══════════════════════════════════════════════════════════════════════
elif menu == "📄 모의고사 지문":
    st.markdown('<div class="page-title">📄 모의고사 지문</div>', unsafe_allow_html=True)
    mocks = load(MOCK_F, [])
    if is_teacher:
        with st.expander("➕ 지문 추가"):
            st.markdown('<div class="panel">', unsafe_allow_html=True)
            pdf_m = st.file_uploader("PDF 업로드", type=["pdf"], key="mock_pdf")
            ext_m = ""
            if pdf_m and PDF_OK:
                r2 = PdfReader(pdf_m); ext_m=" ".join(pg.extract_text() or "" for pg in r2.pages).strip()
                if ext_m: st.success(f"✅ {len(ext_m)}자 추출")
            with st.form("mock_form"):
                c1,c2,c3 = st.columns(3)
                with c1: m_year  = st.text_input("연도","2024")
                with c2: m_month = st.selectbox("시행월",["3월","4월","6월","7월","9월","10월","11월","수능"])
                with c3: m_num   = st.text_input("번호","18")
                m_title    = st.text_input("주제/유형", placeholder="빈칸 추론")
                m_text     = st.text_area("지문 본문", value=ext_m, height=180)
                m_analysis = st.text_area("선생님 해설 (선택)", height=80)
                if st.form_submit_button("💾 저장") and m_year and m_text:
                    mocks.append({"id":f"mock_{len(mocks)+1}","year":m_year,"month":m_month,
                                  "number":m_num,"title":m_title,"text":m_text,"analysis":m_analysis})
                    save(MOCK_F,mocks); st.success("저장!"); st.rerun()
            st.markdown('</div>', unsafe_allow_html=True)

    if not mocks: st.info("등록된 지문 없음")
    else:
        years=["전체"]+sorted(set(p["year"] for p in mocks),reverse=True)
        sy=st.selectbox("연도",years)
        fm=mocks if sy=="전체" else [p for p in mocks if p["year"]==sy]
        for p in fm:
            with st.expander(f"📄 {p['year']} {p['month']} {p.get('number','')}번 — {p.get('title','')}"):
                st.markdown(f'<div class="exam-box">{p["text"]}</div>', unsafe_allow_html=True)
                if p.get("analysis"):
                    st.markdown(f'<div class="panel" style="border-left:3px solid #F59E0B;margin-top:8px;"><span class="badge b-amber">선생님 해설</span><br><br>{p["analysis"]}</div>', unsafe_allow_html=True)

# ══════════════════════════════════════════════════════════════════════
# 본문 변형문제
# ══════════════════════════════════════════════════════════════════════
elif menu == "📖 본문 변형문제":
    st.markdown('<div class="page-title">📖 본문 변형문제</div>', unsafe_allow_html=True)
    var_p = load(VAR_F, [])
    api_key = st.session_state.api_key
    if is_teacher:
        with st.expander("➕ 변형문제 추가"):
            tab_a, tab_m = st.tabs(["🤖 AI 자동 생성","✏️ 직접 등록"])
            with tab_a:
                if not api_key: st.info("🔑 자료 업로드 탭에서 API 키 입력 후 사용 가능")
                else:
                    with st.form("var_auto"):
                        src = st.selectbox("원본 지문", [p["title"] for p in get_passages()])
                        vt  = st.multiselect("유형", ["순서배열","문장삽입","빈칸추론","어법판단","내용일치"], default=["빈칸추론"])
                        if st.form_submit_button("🤖 생성"):
                            sp = next((p for p in get_passages() if p["title"]==src), None)
                            if sp:
                                with st.spinner("생성 중..."):
                                    try:
                                        import anthropic
                                        cl=anthropic.Anthropic(api_key=api_key)
                                        msg=cl.messages.create(model="claude-haiku-4-5-20251001",max_tokens=1500,
                                            messages=[{"role":"user","content":f"지문으로 고등학교 내신 변형문제 생성. 유형:{','.join(vt)}\nJSON만:[{{\"type\":\"유형\",\"question\":\"문제\",\"answer\":\"정답\",\"explanation\":\"해설\"}}]\n\n{sp['full_text']}"}])
                                        raw=msg.content[0].text.strip()
                                        m=re.search(r'\[.*\]',raw,re.DOTALL)
                                        if m:
                                            probs=json.loads(m.group())
                                            for pr in probs: pr["source"]=src; pr["id"]=f"var_{len(var_p)+1}"; var_p.append(pr)
                                            save(VAR_F,var_p); st.success(f"✅ {len(probs)}개 생성!"); st.rerun()
                                    except Exception as e: st.error(str(e))
            with tab_m:
                with st.form("var_manual"):
                    vs=st.text_input("출처"); vt2=st.selectbox("유형",["순서배열","문장삽입","빈칸추론","어법판단","내용일치","기타"])
                    vq=st.text_area("문제",height=140); va=st.text_area("정답/해설",height=70)
                    if st.form_submit_button("💾 저장") and vq:
                        var_p.append({"id":f"var_{len(var_p)+1}","source":vs,"type":vt2,"question":vq,"answer":va})
                        save(VAR_F,var_p); st.success("저장!"); st.rerun()

    if not var_p: st.info("등록된 변형문제 없음")
    else:
        tf=["전체"]+list(set(p.get("type","") for p in var_p))
        st_=st.selectbox("유형 필터",tf)
        fv=var_p if st_=="전체" else [p for p in var_p if p.get("type")==st_]
        for i,pr in enumerate(fv):
            with st.expander(f'[{pr.get("type","")}] {pr.get("source","")} — 문제 {i+1}'):
                st.markdown(f'<div class="exam-box">{pr["question"]}</div>', unsafe_allow_html=True)
                if st.button("정답 보기 👁", key=f"va_{i}"):
                    st.session_state[f"sv_{i}"]=not st.session_state.get(f"sv_{i}",False)
                if st.session_state.get(f"sv_{i}"):
                    st.markdown(f'<div class="result-ok">✅ {pr.get("answer","")}</div>', unsafe_allow_html=True)

# ══════════════════════════════════════════════════════════════════════
# 모의고사 변형문제
# ══════════════════════════════════════════════════════════════════════
elif menu == "🧪 모의고사 변형문제":
    st.markdown('<div class="page-title">🧪 모의고사 변형문제</div>', unsafe_allow_html=True)
    mv_p=load(MVAR_F,[]); mocks=load(MOCK_F,[]); api_key=st.session_state.api_key
    if is_teacher:
        with st.expander("➕ 변형문제 추가"):
            tab_a2,tab_m2=st.tabs(["🤖 AI 자동 생성","✏️ 직접 등록"])
            with tab_a2:
                if not api_key: st.info("🔑 API 키 필요")
                elif not mocks: st.info("모의고사 지문을 먼저 추가하세요")
                else:
                    with st.form("mv_auto"):
                        sm=st.selectbox("모의고사 지문",[f"{p['year']} {p['month']} {p.get('number','')}번" for p in mocks])
                        mt=st.multiselect("유형",["빈칸추론","순서배열","문장삽입","어법판단","주제파악"],default=["빈칸추론"])
                        if st.form_submit_button("🤖 생성"):
                            idx_=[f"{p['year']} {p['month']} {p.get('number','')}번" for p in mocks].index(sm)
                            sp2=mocks[idx_]
                            with st.spinner("생성 중..."):
                                try:
                                    import anthropic
                                    cl=anthropic.Anthropic(api_key=api_key)
                                    msg=cl.messages.create(model="claude-haiku-4-5-20251001",max_tokens=1500,
                                        messages=[{"role":"user","content":f"수능/모의고사 변형문제. 유형:{','.join(mt)}\nJSON만:[{{\"type\":\"유형\",\"question\":\"문제\",\"answer\":\"정답\",\"explanation\":\"해설\"}}]\n\n{sp2['text']}"}])
                                    raw=msg.content[0].text.strip()
                                    m=re.search(r'\[.*\]',raw,re.DOTALL)
                                    if m:
                                        probs=json.loads(m.group())
                                        for pr in probs: pr["source"]=sm; pr["id"]=f"mv_{len(mv_p)+1}"; mv_p.append(pr)
                                        save(MVAR_F,mv_p); st.success(f"✅ {len(probs)}개 생성!"); st.rerun()
                                except Exception as e: st.error(str(e))
            with tab_m2:
                with st.form("mv_manual"):
                    ms2=st.text_input("출처"); mt2=st.selectbox("유형",["빈칸추론","순서배열","문장삽입","어법판단","주제파악","기타"])
                    mq=st.text_area("문제",height=140); ma=st.text_area("정답/해설",height=70)
                    if st.form_submit_button("💾 저장") and mq:
                        mv_p.append({"id":f"mv_{len(mv_p)+1}","source":ms2,"type":mt2,"question":mq,"answer":ma})
                        save(MVAR_F,mv_p); st.success("저장!"); st.rerun()

    if not mv_p: st.info("등록된 모의고사 변형문제 없음")
    else:
        mf=["전체"]+list(set(p.get("type","") for p in mv_p))
        sm2=st.selectbox("유형 필터",mf)
        fmv=mv_p if sm2=="전체" else [p for p in mv_p if p.get("type")==sm2]
        for i,pr in enumerate(fmv):
            with st.expander(f'[{pr.get("type","")}] {pr.get("source","")} — 문제 {i+1}'):
                st.markdown(f'<div class="exam-box">{pr["question"]}</div>', unsafe_allow_html=True)
                if st.button("정답 보기 👁", key=f"mva_{i}"):
                    st.session_state[f"smv_{i}"]=not st.session_state.get(f"smv_{i}",False)
                if st.session_state.get(f"smv_{i}"):
                    st.markdown(f'<div class="result-ok">✅ {pr.get("answer","")}</div>', unsafe_allow_html=True)

# ══════════════════════════════════════════════════════════════════════
# 학생 관리 (선생님)
# ══════════════════════════════════════════════════════════════════════
elif menu == "👥 학생 관리" and is_teacher:
    st.markdown('<div class="page-title">👥 학생 관리</div>', unsafe_allow_html=True)
    users_all = load(USERS_F, {})
    students  = {k:v for k,v in users_all.items() if v["role"]=="student"}

    with st.expander("➕ 학생 추가"):
        st.markdown('<div class="panel">', unsafe_allow_html=True)
        tab_s, tab_b = st.tabs(["개별 추가","CSV 일괄"])
        with tab_s:
            with st.form("add_s"):
                c1,c2,c3=st.columns(3)
                with c1: nid=st.text_input("아이디")
                with c2: nm=st.text_input("이름")
                with c3: npw=st.text_input("비밀번호")
                if st.form_submit_button("추가"):
                    if nid and nm and npw:
                        if nid in users_all: st.error("이미 존재")
                        else:
                            users_all[nid]={"password":hp(npw),"role":"student","name":nm}
                            save(USERS_F,users_all); st.success(f"✅ {nm} 추가!"); st.rerun()
        with tab_b:
            st.code("id,name,password\nstudent01,홍길동,pass1234")
            bf=st.file_uploader("CSV",type=["csv"],key="bulk_s")
            if bf:
                df=pd.read_csv(bf); added=0
                for _,row in df.iterrows():
                    uid2=str(row["id"])
                    if uid2 not in users_all:
                        users_all[uid2]={"password":hp(str(row["password"])),"role":"student","name":str(row["name"])}; added+=1
                save(USERS_F,users_all); st.success(f"✅ {added}명 추가!"); st.rerun()
        st.markdown('</div>', unsafe_allow_html=True)

    st.markdown(f"#### 학생 목록 ({len(students)}명)")
    results=load(RESULTS_F,{})
    for uid2,info in students.items():
        my_r=results.get(uid2,[]); avg2=int(sum(r["score"] for r in my_r)/len(my_r)) if my_r else 0
        c1,c2,c3,c4=st.columns([2,2,1,1])
        c1.markdown(f'<p style="margin:8px 0;color:#1E293B;font-weight:600;">{info["name"]} <span style="color:#9CA3AF;font-weight:400;">@{uid2}</span></p>', unsafe_allow_html=True)
        c2.markdown(f'<p style="margin:8px 0;color:#6B7280;font-size:12px;">응시 {len(my_r)}회 · 평균 {avg2}점</p>', unsafe_allow_html=True)
        c3.markdown('<span class="badge b-blue">학생</span>', unsafe_allow_html=True)
        with c4:
            if st.button("삭제", key=f"del_{uid2}"):
                del users_all[uid2]; save(USERS_F,users_all); st.rerun()
        st.markdown('<hr style="margin:4px 0;">', unsafe_allow_html=True)

# ══════════════════════════════════════════════════════════════════════
# 성적 관리 (선생님)
# ══════════════════════════════════════════════════════════════════════
elif menu == "📊 성적 관리" and is_teacher:
    st.markdown('<div class="page-title">📊 성적 관리</div>', unsafe_allow_html=True)
    users_all=load(USERS_F,{}); results=load(RESULTS_F,{})
    students={k:v for k,v in users_all.items() if v["role"]=="student"}
    tasks=get_tasks()

    tab1,tab2=st.tabs(["전체 성적","과제별 성적"])
    with tab1:
        rows=[]
        for uid2,rlist in results.items():
            if uid2 not in students: continue
            nm=students[uid2]["name"]
            for r in rlist:
                rows.append({"이름":nm,"날짜":r["date"],"지문":r["passage_title"],"점수(%)":r["score"],"정답":r["correct"],"전체":r["total"]})
        if rows:
            df=pd.DataFrame(rows)
            passages_titles=["전체"]+list(df["지문"].unique())
            sp=st.selectbox("지문 필터",passages_titles)
            if sp!="전체": df=df[df["지문"]==sp]
            st.dataframe(df.sort_values("날짜",ascending=False),use_container_width=True,hide_index=True)
            st.markdown("---")
            st.markdown("#### 학생별 평균")
            summary=df.groupby("이름")["점수(%)"].agg(["mean","count"]).reset_index()
            summary.columns=["이름","평균","응시수"]; summary=summary.sort_values("평균",ascending=False)
            for _,row in summary.iterrows():
                pct=round(row["평균"],1)
                color="#15803D" if pct>=80 else "#A16207" if pct>=60 else "#BE123C"
                st.markdown(f'<div style="display:flex;align-items:center;gap:12px;margin:6px 0;"><span style="min-width:80px;font-weight:600;color:#1E293B;">{row["이름"]}</span><span style="color:{color};font-weight:700;min-width:50px;">{pct}점</span><span style="color:#9CA3AF;font-size:12px;">{int(row["응시수"])}회</span></div><div class="progress-bg"><div class="progress-fill" style="width:{pct}%;background:{color};"></div></div>', unsafe_allow_html=True)
            csv=df.to_csv(index=False,encoding="utf-8-sig")
            st.download_button("📥 CSV 다운로드", csv, "성적표.csv","text/csv")
        else: st.info("시험 기록 없음")

    with tab2:
        if not tasks: st.info("과제 없음")
        else:
            sel_t=st.selectbox("과제 선택",[t["title"] for t in tasks])
            t_obj=next((t for t in tasks if t["title"]==sel_t),None)
            if t_obj:
                rows2=[]
                for uid2,info in students.items():
                    t_res=[r for r in results.get(uid2,[]) if r.get("task_id")==t_obj["id"]]
                    if t_res: rows2.append({"이름":info["name"],"완료":"✅","최고점":max(r["score"] for r in t_res),"응시":len(t_res)})
                    else:      rows2.append({"이름":info["name"],"완료":"❌","최고점":"-","응시":0})
                st.dataframe(pd.DataFrame(rows2),use_container_width=True,hide_index=True)

# ══════════════════════════════════════════════════════════════════════
# 내 성적 (학생)
# ══════════════════════════════════════════════════════════════════════
elif menu == "📊 내 성적":
    st.markdown('<div class="page-title">📊 내 성적</div>', unsafe_allow_html=True)
    uid=st.session_state.username; results=load(RESULTS_F,{}); my_res=results.get(uid,[])
    if not my_res: st.info("아직 시험 기록이 없어요. 본문 암기 시험을 응시해봐요!")
    else:
        avg=int(sum(r["score"] for r in my_res)/len(my_res)); best=max(r["score"] for r in my_res)
        c1,c2,c3=st.columns(3)
        c1.markdown(f'<div class="stat-card"><div class="stat-icon si-blue">📝</div><div class="stat-val">{len(my_res)}</div><div class="stat-label">응시 횟수</div></div>', unsafe_allow_html=True)
        c2.markdown(f'<div class="stat-card"><div class="stat-icon si-amber">📊</div><div class="stat-val">{avg}%</div><div class="stat-label">평균 점수</div></div>', unsafe_allow_html=True)
        c3.markdown(f'<div class="stat-card"><div class="stat-icon si-green">🏆</div><div class="stat-val">{best}%</div><div class="stat-label">최고 점수</div></div>', unsafe_allow_html=True)
        st.markdown("---")
        df=pd.DataFrame(my_res)[["date","passage_title","score","correct","total"]]
        df.columns=["날짜","지문","점수(%)","정답","전체"]
        st.dataframe(df[::-1],use_container_width=True,hide_index=True)
