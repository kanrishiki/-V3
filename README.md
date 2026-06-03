import React, { useMemo, useState } from "react";
import {
  Search,
  Play,
  Square,
  Clock,
  Download,
  Activity,
  CalendarDays,
  Users,
  UserRoundPlus,
  Pencil,
  Trash2,
  Save,
  X,
  LogOut,
} from "lucide-react";
import { Card, CardContent } from "@/components/ui/card";
import { Button } from "@/components/ui/button";

const FACILITIES = [
  { id: "kamagaya", name: "ウェルホーム+ナースステーション鎌ヶ谷", short: "鎌ヶ谷" },
  { id: "kawaguchi", name: "ウェルホーム+ナースステーション川口", short: "川口" },
  { id: "shiki", name: "ウェルホーム+ナースステーション志木", short: "志木" },
  { id: "kanazawa", name: "ウェルホーム+ナースステーション金沢", short: "金沢" },
  { id: "ichinomiya", name: "ウェルホーム+ナースステーション一宮", short: "一宮" },
];

const DEMO_PASSWORD = "demo1234";

export default function NursingTimeRecordApp() {
  const nursingOptions = [
    "バイタル測定",
    "服薬確認・管理",
    "処置",
    "創部観察",
    "点滴管理",
    "吸引",
    "経管栄養",
    "排泄ケア",
    "入浴前後の健康確認",
    "リハビリ・機能訓練補助",
    "状態観察",
    "家族・医師への連絡",
    "看護記録作成",
    "その他",
  ];

  const today = new Date().toISOString().slice(0, 10);

  const [loginFacilityId, setLoginFacilityId] = useState(FACILITIES[0].id);
  const [password, setPassword] = useState("");
  const [currentFacility, setCurrentFacility] = useState(null);

  const [users, setUsers] = useState([
    { id: 1, facilityId: "kamagaya", name: "山田 太郎", room: "101", status: "利用中" },
    { id: 2, facilityId: "kamagaya", name: "佐藤 花子", room: "102", status: "利用中" },
    { id: 3, facilityId: "kawaguchi", name: "川口 利用者A", room: "201", status: "利用中" },
    { id: 4, facilityId: "shiki", name: "志木 利用者A", room: "301", status: "利用中" },
    { id: 5, facilityId: "kanazawa", name: "金沢 利用者A", room: "401", status: "利用中" },
    { id: 6, facilityId: "ichinomiya", name: "一宮 利用者A", room: "501", status: "利用中" },
  ]);

  const [nurses, setNurses] = useState([
    { id: 1, facilityId: "kamagaya", name: "西川", role: "施設長" },
    { id: 2, facilityId: "kamagaya", name: "看護師A", role: "看護師" },
    { id: 3, facilityId: "kawaguchi", name: "川口 看護師A", role: "看護師" },
    { id: 4, facilityId: "shiki", name: "志木 看護師A", role: "看護師" },
    { id: 5, facilityId: "kanazawa", name: "金沢 看護師A", role: "看護師" },
    { id: 6, facilityId: "ichinomiya", name: "一宮 看護師A", role: "看護師" },
  ]);

  const [records, setRecords] = useState([
    {
      id: 1,
      facilityId: "kamagaya",
      date: today,
      visitNo: 1,
      user: "山田 太郎",
      nurse: "西川",
      content: "バイタル測定",
      startTime: "09:00",
      endTime: "09:25",
      minutes: 25,
      vitals: {
        temperature: "36.5",
        systolic: "128",
        diastolic: "74",
        pulse: "72",
        spo2: "98",
        memo: "状態安定。服薬確認済み。",
      },
    },
  ]);

  const [newUser, setNewUser] = useState({ name: "", room: "", status: "利用中" });
  const [newNurse, setNewNurse] = useState({ name: "", role: "看護師" });

  const [selectedUser, setSelectedUser] = useState("");
  const [userSearch, setUserSearch] = useState("");
  const [selectedNurse, setSelectedNurse] = useState("");
  const [recordDate, setRecordDate] = useState(today);
  const [nursingContent, setNursingContent] = useState("バイタル測定");
  const [customContent, setCustomContent] = useState("");
  const [activeVisit, setActiveVisit] = useState(null);
  const [editingId, setEditingId] = useState(null);
  const [editRecord, setEditRecord] = useState(null);
  const [achievementTab, setAchievementTab] = useState("日別");
  const [achievementUser, setAchievementUser] = useState("全員");

  const [vitals, setVitals] = useState({
    temperature: "",
    systolic: "",
    diastolic: "",
    pulse: "",
    spo2: "",
    memo: "",
  });

  const scopedUsers = users.filter((u) => u.facilityId === currentFacility?.id);
  const scopedNurses = nurses.filter((n) => n.facilityId === currentFacility?.id);
  const scopedRecords = records.filter((r) => r.facilityId === currentFacility?.id);

  const filteredUsers = scopedUsers.filter(
    (user) => user.name.includes(userSearch) || user.room.includes(userSearch)
  );

  const formatTime = (date) => date.toTimeString().slice(0, 5);

  const calcMinutes = (start, end) => {
    if (!start || !end || !start.includes(":") || !end.includes(":")) return 0;
    const [sh, sm] = start.split(":").map(Number);
    const [eh, em] = end.split(":").map(Number);
    return Math.max(0, eh * 60 + em - (sh * 60 + sm));
  };

  const formatMinutes = (minutes) => {
    return `${Math.floor(minutes / 60)}時間 ${minutes % 60}分`;
  };

  const getWeekRange = (dateString) => {
    const base = new Date(dateString);
    const day = base.getDay();
    const diffToMonday = day === 0 ? -6 : 1 - day;

    const monday = new Date(base);
    monday.setDate(base.getDate() + diffToMonday);

    const sunday = new Date(monday);
    sunday.setDate(monday.getDate() + 6);

    const toYmd = (d) => d.toISOString().slice(0, 10);

    return {
      start: toYmd(monday),
      end: toYmd(sunday),
    };
  };

  const login = () => {
    if (password !== DEMO_PASSWORD) {
      alert("パスワードが違います。試作版は demo1234 です。");
      return;
    }

    const facility = FACILITIES.find((f) => f.id === loginFacilityId);
    setCurrentFacility(facility);

    const firstNurse = nurses.find((n) => n.facilityId === facility.id);
    const firstUser = users.find((u) => u.facilityId === facility.id);

    setSelectedNurse(firstNurse?.name || "");
    setSelectedUser(firstUser?.name || "");
  };

  const logout = () => {
    setCurrentFacility(null);
    setPassword("");
    setSelectedUser("");
    setSelectedNurse("");
    setActiveVisit(null);
  };

  const addUser = () => {
    if (!newUser.name.trim()) return;

    setUsers([
      ...users,
      {
        id: Date.now(),
        facilityId: currentFacility.id,
        ...newUser,
        name: newUser.name.trim(),
      },
    ]);

    setNewUser({ name: "", room: "", status: "利用中" });
  };

  const deleteUser = (id, name) => {
    if (!confirm(`${name} 様を削除しますか？`)) return;

    setUsers(users.filter((user) => user.id !== id));
    if (selectedUser === name) setSelectedUser("");
  };

  const addNurse = () => {
    if (!newNurse.name.trim()) return;

    setNurses([
      ...nurses,
      {
        id: Date.now(),
        facilityId: currentFacility.id,
        ...newNurse,
        name: newNurse.name.trim(),
      },
    ]);

    setNewNurse({ name: "", role: "看護師" });
  };

  const deleteNurse = (id, name) => {
    if (!confirm(`${name} を削除しますか？`)) return;

    setNurses(nurses.filter((nurse) => nurse.id !== id));
    if (selectedNurse === name) setSelectedNurse("");
  };

  const startVisit = () => {
    if (!selectedUser) return alert("利用者様を選択してください。");
    if (!selectedNurse) return alert("看護師を選択してください。");

    const now = new Date();

    setActiveVisit({
      facilityId: currentFacility.id,
      user: selectedUser,
      nurse: selectedNurse,
      date: recordDate,
      content: nursingContent === "その他" ? customContent || "その他" : nursingContent,
      startTime: formatTime(now),
    });
  };

  const endVisit = () => {
    if (!activeVisit) return;

    const now = new Date();
    const endTime = formatTime(now);

    const visitNo =
      scopedRecords.filter((r) => r.date === activeVisit.date && r.user === activeVisit.user)
        .length + 1;

    setRecords([
      ...records,
      {
        id: Date.now(),
        ...activeVisit,
        visitNo,
        endTime,
        minutes: calcMinutes(activeVisit.startTime, endTime),
        vitals,
      },
    ]);

    setActiveVisit(null);
    setVitals({ temperature: "", systolic: "", diastolic: "", pulse: "", spo2: "", memo: "" });
  };

  const beginEdit = (record) => {
    setEditingId(record.id);
    setEditRecord(JSON.parse(JSON.stringify(record)));
  };

  const cancelEdit = () => {
    setEditingId(null);
    setEditRecord(null);
  };

  const saveEdit = () => {
    if (!editRecord) return;

    const minutes = calcMinutes(editRecord.startTime, editRecord.endTime);

    setRecords(
      records.map((record) =>
        record.id === editingId ? { ...editRecord, minutes } : record
      )
    );

    cancelEdit();
  };

  const deleteRecord = (id) => {
    if (!confirm("この記録を削除しますか？")) return;
    setRecords(records.filter((record) => record.id !== id));
  };

  const dailyUserTotals = useMemo(() => {
    const target = scopedRecords.filter((r) => r.date === recordDate);

    return target.reduce((acc, r) => {
      if (!acc[r.user]) acc[r.user] = { visits: 0, minutes: 0 };
      acc[r.user].visits += 1;
      acc[r.user].minutes += r.minutes;
      return acc;
    }, {});
  }, [scopedRecords, recordDate]);

  const monthlyRecords = useMemo(() => {
    const month = recordDate.slice(0, 7);
    return scopedRecords.filter((r) => r.date.startsWith(month));
  }, [scopedRecords, recordDate]);

  const monthlyUserTotals = useMemo(() => {
    return monthlyRecords.reduce((acc, r) => {
      if (!acc[r.user]) acc[r.user] = { visits: 0, minutes: 0 };
      acc[r.user].visits += 1;
      acc[r.user].minutes += r.minutes;
      return acc;
    }, {});
  }, [monthlyRecords]);

  const totalMonthlyMinutes = monthlyRecords.reduce((sum, r) => sum + r.minutes, 0);

  const userAchievementRecords = useMemo(() => {
    if (achievementUser === "全員") return scopedRecords;
    return scopedRecords.filter((r) => r.user === achievementUser);
  }, [scopedRecords, achievementUser]);

  const achievementDaily = useMemo(() => {
    return userAchievementRecords.reduce((acc, r) => {
      if (!acc[r.date]) acc[r.date] = { visits: 0, minutes: 0 };
      acc[r.date].visits += 1;
      acc[r.date].minutes += r.minutes;
      return acc;
    }, {});
  }, [userAchievementRecords]);

  const achievementWeekly = useMemo(() => {
    return userAchievementRecords.reduce((acc, r) => {
      const range = getWeekRange(r.date);
      const key = `${range.start}〜${range.end}`;

      if (!acc[key]) acc[key] = { visits: 0, minutes: 0 };
      acc[key].visits += 1;
      acc[key].minutes += r.minutes;

      return acc;
    }, {});
  }, [userAchievementRecords]);

  const achievementMonthly = useMemo(() => {
    return userAchievementRecords.reduce((acc, r) => {
      const month = r.date.slice(0, 7);

      if (!acc[month]) acc[month] = { visits: 0, minutes: 0 };
      acc[month].visits += 1;
      acc[month].minutes += r.minutes;

      return acc;
    }, {});
  }, [userAchievementRecords]);

  const achievementByUser = useMemo(() => {
    return scopedRecords.reduce((acc, r) => {
      if (!acc[r.user]) acc[r.user] = { visits: 0, minutes: 0 };
      acc[r.user].visits += 1;
      acc[r.user].minutes += r.minutes;
      return acc;
    }, {});
  }, [scopedRecords]);

  const createCsv = (targetRecords, fileName) => {
    const header = [
      "拠点",
      "日付",
      "訪問回数",
      "利用者名",
      "看護師名",
      "看護内容",
      "開始時間",
      "終了時間",
      "合計分",
      "体温",
      "血圧上",
      "血圧下",
      "脈拍",
      "SpO2",
      "メモ",
    ];

    const rows = targetRecords.map((r) => [
      currentFacility.name,
      r.date,
      r.visitNo,
      r.user,
      r.nurse,
      r.content,
      r.startTime,
      r.endTime,
      r.minutes,
      r.vitals.temperature,
      r.vitals.systolic,
      r.vitals.diastolic,
      r.vitals.pulse,
      r.vitals.spo2,
      r.vitals.memo,
    ]);

    const csv = [header, ...rows]
      .map((row) => row.map((cell) => `"${String(cell ?? "").replaceAll('"', '""')}"`).join(","))
      .join("\n");

    const blob = new Blob(["\uFEFF" + csv], { type: "text/csv;charset=utf-8;" });
    const url = URL.createObjectURL(blob);
    const a = document.createElement("a");

    a.href = url;
    a.download = fileName;
    a.click();

    URL.revokeObjectURL(url);
  };

  const downloadDailyCSV = () => {
    const target = scopedRecords.filter((r) => r.date === recordDate);
    createCsv(target, `日別CSV_${currentFacility.short}_${recordDate}.csv`);
  };

  const downloadWeeklyCSV = () => {
    const range = getWeekRange(recordDate);
    const target = scopedRecords.filter((r) => r.date >= range.start && r.date <= range.end);
    createCsv(target, `週別CSV_${currentFacility.short}_${range.start}_${range.end}.csv`);
  };

  const downloadMonthlyCSV = () => {
    const month = recordDate.slice(0, 7);
    const target = scopedRecords.filter((r) => r.date.startsWith(month));
    createCsv(target, `月別CSV_${currentFacility.short}_${month}.csv`);
  };

  const downloadUserCSV = () => {
    if (achievementUser === "全員") {
      alert("利用者別CSVを出力する場合は、利用者様を選択してください。");
      return;
    }

    const target = scopedRecords.filter((r) => r.user === achievementUser);
    createCsv(target, `利用者別CSV_${currentFacility.short}_${achievementUser}.csv`);
  };

  const TimeText = ({ minutes }) => (
    <span>
      {Math.floor(minutes / 60)}時間 {minutes % 60}分
    </span>
  );

  if (!currentFacility) {
    return (
      <div className="min-h-screen bg-gray-100 flex items-center justify-center p-4">
        <Card className="w-full max-w-md rounded-2xl shadow-sm">
          <CardContent className="p-6 space-y-5">
            <div>
              <h1 className="text-2xl font-bold">看護記録・時間積算アプリ</h1>
              <p className="text-sm text-gray-600 mt-2">
                拠点アカウントでログインしてください。
              </p>
            </div>

            <div>
              <label className="text-sm text-gray-600">ログイン拠点</label>
              <select
                className="w-full rounded-xl border p-3 bg-white"
                value={loginFacilityId}
                onChange={(e) => setLoginFacilityId(e.target.value)}
              >
                {FACILITIES.map((facility) => (
                  <option key={facility.id} value={facility.id}>
                    {facility.name}
                  </option>
                ))}
              </select>
            </div>

            <div>
              <label className="text-sm text-gray-600">パスワード</label>
              <input
                type="password"
                className="w-full rounded-xl border p-3 bg-white"
                placeholder="試作版：demo1234"
                value={password}
                onChange={(e) => setPassword(e.target.value)}
              />
            </div>

            <Button onClick={login} className="w-full rounded-xl py-6 text-base">
              ログイン
            </Button>

            <p className="text-xs text-gray-500">
              ※本番運用ではSupabase等で個別ID・パスワード管理を行ってください。
            </p>
          </CardContent>
        </Card>
      </div>
    );
  }

  return (
    <div className="min-h-screen bg-gray-100 p-4 md:p-8 text-gray-900">
      <div className="max-w-7xl mx-auto space-y-6">
        <div className="flex flex-col md:flex-row md:items-end md:justify-between gap-4">
          <div>
            <h1 className="text-3xl font-bold">看護記録・時間積算アプリ</h1>
            <p className="text-gray-600 mt-1">
              {currentFacility.name}
            </p>
            <p className="text-sm text-gray-500">
              ログイン中の拠点データのみ表示・編集・CSV出力します。
            </p>
          </div>

          <div className="flex flex-wrap gap-2">
            <Button onClick={downloadDailyCSV} className="rounded-xl">
              <Download className="w-4 h-4 mr-2" />
              日別CSV
            </Button>

            <Button onClick={downloadWeeklyCSV} className="rounded-xl">
              <Download className="w-4 h-4 mr-2" />
              週別CSV
            </Button>

            <Button onClick={downloadMonthlyCSV} className="rounded-xl">
              <Download className="w-4 h-4 mr-2" />
              月別CSV
            </Button>

            <Button onClick={downloadUserCSV} className="rounded-xl">
              <Download className="w-4 h-4 mr-2" />
              利用者別CSV
            </Button>

            <Button onClick={logout} variant="outline" className="rounded-xl">
              <LogOut className="w-4 h-4 mr-2" />
              ログアウト
            </Button>
          </div>
        </div>

        <div className="grid grid-cols-1 xl:grid-cols-3 gap-6">
          <Card className="rounded-2xl shadow-sm xl:col-span-2">
            <CardContent className="p-6 space-y-5">
              <div className="flex items-center gap-2 font-bold text-xl">
                <Clock className="w-5 h-5" />
                訪問記録入力
              </div>

              <div className="grid grid-cols-1 md:grid-cols-3 gap-3">
                <div>
                  <label className="text-sm text-gray-600">日付自動入力</label>
                  <input
                    type="date"
                    className="w-full rounded-xl border p-3 bg-white"
                    value={recordDate}
                    onChange={(e) => setRecordDate(e.target.value)}
                  />
                </div>

                <div>
                  <label className="text-sm text-gray-600">看護師名</label>
                  <select
                    className="w-full rounded-xl border p-3 bg-white"
                    value={selectedNurse}
                    onChange={(e) => setSelectedNurse(e.target.value)}
                  >
                    <option value="">選択してください</option>
                    {scopedNurses.map((nurse) => (
                      <option key={nurse.id} value={nurse.name}>
                        {nurse.name}
                      </option>
                    ))}
                  </select>
                </div>

                <div>
                  <label className="text-sm text-gray-600">看護内容</label>
                  <select
                    className="w-full rounded-xl border p-3 bg-white"
                    value={nursingContent}
                    onChange={(e) => setNursingContent(e.target.value)}
                  >
                    {nursingOptions.map((option) => (
                      <option key={option} value={option}>
                        {option}
                      </option>
                    ))}
                  </select>
                </div>
              </div>

              {nursingContent === "その他" && (
                <input
                  className="w-full rounded-xl border p-3 bg-white"
                  placeholder="その他の看護内容"
                  value={customContent}
                  onChange={(e) => setCustomContent(e.target.value)}
                />
              )}

              <div>
                <label className="text-sm text-gray-600">利用者検索</label>
                <div className="relative">
                  <Search className="w-4 h-4 absolute left-3 top-4 text-gray-500" />
                  <input
                    className="w-full rounded-xl border p-3 pl-10 bg-white"
                    placeholder="利用者名・居室番号を検索"
                    value={userSearch}
                    onChange={(e) => setUserSearch(e.target.value)}
                  />
                </div>

                <div className="flex flex-wrap gap-2 mt-3">
                  {filteredUsers.map((user) => (
                    <button
                      key={user.id}
                      onClick={() => setSelectedUser(user.name)}
                      className={`rounded-full px-4 py-2 border ${
                        selectedUser === user.name ? "bg-gray-900 text-white" : "bg-white"
                      }`}
                    >
                      {user.room && `${user.room}号 `}
                      {user.name}
                    </button>
                  ))}
                </div>
              </div>

              <div className="grid grid-cols-1 md:grid-cols-2 gap-3">
                <Button
                  onClick={startVisit}
                  disabled={!selectedUser || !!activeVisit}
                  className="rounded-xl py-6 text-base"
                >
                  <Play className="w-5 h-5 mr-2" />
                  開始
                </Button>

                <Button
                  onClick={endVisit}
                  disabled={!activeVisit}
                  variant="outline"
                  className="rounded-xl py-6 text-base"
                >
                  <Square className="w-5 h-5 mr-2" />
                  終了
                </Button>
              </div>

              <div className="grid grid-cols-1 md:grid-cols-5 gap-3">
                <input
                  type="number"
                  step="0.1"
                  placeholder="体温"
                  className="rounded-xl border p-3 bg-white"
                  value={vitals.temperature}
                  onChange={(e) => setVitals({ ...vitals, temperature: e.target.value })}
                />

                <input
                  placeholder="血圧 上"
                  className="rounded-xl border p-3 bg-white"
                  value={vitals.systolic}
                  onChange={(e) => setVitals({ ...vitals, systolic: e.target.value })}
                />

                <input
                  placeholder="血圧 下"
                  className="rounded-xl border p-3 bg-white"
                  value={vitals.diastolic}
                  onChange={(e) => setVitals({ ...vitals, diastolic: e.target.value })}
                />

                <input
                  placeholder="脈拍"
                  className="rounded-xl border p-3 bg-white"
                  value={vitals.pulse}
                  onChange={(e) => setVitals({ ...vitals, pulse: e.target.value })}
                />

                <input
                  placeholder="SpO2"
                  className="rounded-xl border p-3 bg-white"
                  value={vitals.spo2}
                  onChange={(e) => setVitals({ ...vitals, spo2: e.target.value })}
                />
              </div>

              <textarea
                className="w-full rounded-xl border p-3 bg-white min-h-24"
                placeholder="状態・申し送り・特記事項"
                value={vitals.memo}
                onChange={(e) => setVitals({ ...vitals, memo: e.target.value })}
              />

              {activeVisit && (
                <div className="rounded-xl bg-gray-900 text-white p-4">
                  記録中：{activeVisit.user} / {activeVisit.content} / 開始{" "}
                  {activeVisit.startTime}
                </div>
              )}
            </CardContent>
          </Card>

          <Card className="rounded-2xl shadow-sm">
            <CardContent className="p-6 space-y-5">
              <div className="flex items-center gap-2 font-bold text-xl">
                <CalendarDays className="w-5 h-5" />
                日別・月間別累計
              </div>

              <div className="rounded-2xl bg-gray-50 p-4">
                <p className="text-sm text-gray-500">選択日</p>
                <p className="text-2xl font-bold">{recordDate}</p>
              </div>

              <div>
                <h3 className="font-bold mb-2">日別：利用者様ごとの累積介入時間</h3>
                <div className="space-y-2">
                  {Object.entries(dailyUserTotals).length === 0 ? (
                    <p className="text-sm text-gray-500">この日の記録はありません。</p>
                  ) : (
                    Object.entries(dailyUserTotals).map(([user, s]) => (
                      <div key={user} className="border rounded-xl p-3 bg-white">
                        <p className="font-bold">{user}</p>
                        <p className="text-sm text-gray-600">
                          訪問 {s.visits}回 / <TimeText minutes={s.minutes} />
                        </p>
                      </div>
                    ))
                  )}
                </div>
              </div>

              <div>
                <h3 className="font-bold mb-2">月間：利用者様ごとの累積介入時間</h3>
                <p className="text-sm text-gray-500 mb-2">
                  対象月：{recordDate.slice(0, 7)} / 全体{" "}
                  <TimeText minutes={totalMonthlyMinutes} />
                </p>

                <div className="space-y-2">
                  {Object.entries(monthlyUserTotals).length === 0 ? (
                    <p className="text-sm text-gray-500">この月の記録はありません。</p>
                  ) : (
                    Object.entries(monthlyUserTotals).map(([user, s]) => (
                      <div key={user} className="border rounded-xl p-3 bg-white">
                        <p className="font-bold">{user}</p>
                        <p className="text-sm text-gray-600">
                          訪問 {s.visits}回 / <TimeText minutes={s.minutes} />
                        </p>
                      </div>
                    ))
                  )}
                </div>
              </div>
            </CardContent>
          </Card>
        </div>

        <div className="grid grid-cols-1 xl:grid-cols-2 gap-6">
          <Card className="rounded-2xl shadow-sm">
            <CardContent className="p-6 space-y-4">
              <div className="flex items-center gap-2 font-bold text-xl">
                <Users className="w-5 h-5" />
                利用者マスタ管理
              </div>

              <div className="grid grid-cols-1 md:grid-cols-4 gap-2">
                <input
                  className="rounded-xl border p-3 bg-white md:col-span-2"
                  placeholder="利用者名"
                  value={newUser.name}
                  onChange={(e) => setNewUser({ ...newUser, name: e.target.value })}
                />

                <input
                  className="rounded-xl border p-3 bg-white"
                  placeholder="居室"
                  value={newUser.room}
                  onChange={(e) => setNewUser({ ...newUser, room: e.target.value })}
                />

                <Button onClick={addUser} className="rounded-xl">
                  <UserRoundPlus className="w-4 h-4 mr-2" />
                  追加
                </Button>
              </div>

              <div className="space-y-2 max-h-64 overflow-y-auto">
                {scopedUsers.map((user) => (
                  <div
                    key={user.id}
                    className="flex justify-between items-center rounded-xl border bg-gray-50 p-3"
                  >
                    <div>
                      <p className="font-bold">{user.name}</p>
                      <p className="text-sm text-gray-600">
                        居室：{user.room || "未設定"} / {user.status}
                      </p>
                    </div>

                    <Button
                      variant="ghost"
                      size="icon"
                      onClick={() => deleteUser(user.id, user.name)}
                    >
                      <Trash2 className="w-4 h-4" />
                    </Button>
                  </div>
                ))}
              </div>
            </CardContent>
          </Card>

          <Card className="rounded-2xl shadow-sm">
            <CardContent className="p-6 space-y-4">
              <div className="flex items-center gap-2 font-bold text-xl">
                <Users className="w-5 h-5" />
                看護師マスタ管理
              </div>

              <div className="grid grid-cols-1 md:grid-cols-4 gap-2">
                <input
                  className="rounded-xl border p-3 bg-white md:col-span-2"
                  placeholder="看護師名"
                  value={newNurse.name}
                  onChange={(e) => setNewNurse({ ...newNurse, name: e.target.value })}
                />

                <input
                  className="rounded-xl border p-3 bg-white"
                  placeholder="役職"
                  value={newNurse.role}
                  onChange={(e) => setNewNurse({ ...newNurse, role: e.target.value })}
                />

                <Button onClick={addNurse} className="rounded-xl">
                  <UserRoundPlus className="w-4 h-4 mr-2" />
                  追加
                </Button>
              </div>

              <div className="space-y-2 max-h-64 overflow-y-auto">
                {scopedNurses.map((nurse) => (
                  <div
                    key={nurse.id}
                    className="flex justify-between items-center rounded-xl border bg-gray-50 p-3"
                  >
                    <div>
                      <p className="font-bold">{nurse.name}</p>
                      <p className="text-sm text-gray-600">
                        役職：{nurse.role || "未設定"}
                      </p>
                    </div>

                    <Button
                      variant="ghost"
                      size="icon"
                      onClick={() => deleteNurse(nurse.id, nurse.name)}
                    >
                      <Trash2 className="w-4 h-4" />
                    </Button>
                  </div>
                ))}
              </div>
            </CardContent>
          </Card>
        </div>

        <Card className="rounded-2xl shadow-sm">
          <CardContent className="p-6 space-y-4">
            <div className="flex items-center gap-2 font-bold text-xl">
              <Activity className="w-5 h-5" />
              訪問別集計・記録一覧
            </div>

            <div className="overflow-x-auto">
              <table className="w-full text-sm border-collapse min-w-[1100px]">
                <thead>
                  <tr className="bg-gray-100">
                    <th className="border p-2 text-left">日付</th>
                    <th className="border p-2 text-left">訪問</th>
                    <th className="border p-2 text-left">利用者</th>
                    <th className="border p-2 text-left">看護師</th>
                    <th className="border p-2 text-left">内容</th>
                    <th className="border p-2 text-left">時間</th>
                    <th className="border p-2 text-left">バイタル</th>
                    <th className="border p-2 text-left">メモ</th>
                    <th className="border p-2 text-left">操作</th>
                  </tr>
                </thead>

                <tbody>
                  {scopedRecords.map((r) => (
                    <tr key={r.id}>
                      {editingId === r.id ? (
                        <>
                          <td className="border p-2">
                            <input
                              type="date"
                              className="border rounded p-2 w-full"
                              value={editRecord.date}
                              onChange={(e) =>
                                setEditRecord({ ...editRecord, date: e.target.value })
                              }
                            />
                          </td>

                          <td className="border p-2">{editRecord.visitNo}回目</td>

                          <td className="border p-2">
                            <select
                              className="border rounded p-2 w-full"
                              value={editRecord.user}
                              onChange={(e) =>
                                setEditRecord({ ...editRecord, user: e.target.value })
                              }
                            >
                              {scopedUsers.map((u) => (
                                <option key={u.id} value={u.name}>
                                  {u.name}
                                </option>
                              ))}
                            </select>
                          </td>

                          <td className="border p-2">
                            <select
                              className="border rounded p-2 w-full"
                              value={editRecord.nurse}
                              onChange={(e) =>
                                setEditRecord({ ...editRecord, nurse: e.target.value })
                              }
                            >
                              {scopedNurses.map((n) => (
                                <option key={n.id} value={n.name}>
                                  {n.name}
                                </option>
                              ))}
                            </select>
                          </td>

                          <td className="border p-2">
                            <input
                              className="border rounded p-2 w-full"
                              value={editRecord.content}
                              onChange={(e) =>
                                setEditRecord({ ...editRecord, content: e.target.value })
                              }
                            />
                          </td>

                          <td className="border p-2">
                            <input
                              className="border rounded p-2 w-20"
                              value={editRecord.startTime}
                              onChange={(e) =>
                                setEditRecord({ ...editRecord, startTime: e.target.value })
                              }
                            />
                            〜
                            <input
                              className="border rounded p-2 w-20"
                              value={editRecord.endTime}
                              onChange={(e) =>
                                setEditRecord({ ...editRecord, endTime: e.target.value })
                              }
                            />
                          </td>

                          <td className="border p-2">
                            <input
                              type="number"
                              step="0.1"
                              className="border rounded p-2 w-full"
                              placeholder="体温"
                              value={editRecord.vitals.temperature}
                              onChange={(e) =>
                                setEditRecord({
                                  ...editRecord,
                                  vitals: {
                                    ...editRecord.vitals,
                                    temperature: e.target.value,
                                  },
                                })
                              }
                            />
                          </td>

                          <td className="border p-2">
                            <input
                              className="border rounded p-2 w-full"
                              value={editRecord.vitals.memo}
                              onChange={(e) =>
                                setEditRecord({
                                  ...editRecord,
                                  vitals: { ...editRecord.vitals, memo: e.target.value },
                                })
                              }
                            />
                          </td>

                          <td className="border p-2">
                            <div className="flex gap-1">
                              <Button size="icon" onClick={saveEdit}>
                                <Save className="w-4 h-4" />
                              </Button>

                              <Button size="icon" variant="outline" onClick={cancelEdit}>
                                <X className="w-4 h-4" />
                              </Button>
                            </div>
                          </td>
                        </>
                      ) : (
                        <>
                          <td className="border p-2">{r.date}</td>
                          <td className="border p-2">{r.visitNo}回目</td>
                          <td className="border p-2">{r.user}</td>
                          <td className="border p-2">{r.nurse}</td>
                          <td className="border p-2">{r.content}</td>
                          <td className="border p-2">
                            {r.startTime}〜{r.endTime}
                            <br />
                            {r.minutes}分
                          </td>
                          <td className="border p-2">
                            BT {r.vitals.temperature} / BP {r.vitals.systolic}-
                            {r.vitals.diastolic} / P {r.vitals.pulse} / SpO2{" "}
                            {r.vitals.spo2}
                          </td>
                          <td className="border p-2">{r.vitals.memo}</td>
                          <td className="border p-2">
                            <div className="flex gap-1">
                              <Button
                                size="icon"
                                variant="outline"
                                onClick={() => beginEdit(r)}
                              >
                                <Pencil className="w-4 h-4" />
                              </Button>

                              <Button
                                size="icon"
                                variant="ghost"
                                onClick={() => deleteRecord(r.id)}
                              >
                                <Trash2 className="w-4 h-4" />
                              </Button>
                            </div>
                          </td>
                        </>
                      )}
                    </tr>
                  ))}
                </tbody>
              </table>
            </div>
          </CardContent>
        </Card>

        <Card className="rounded-2xl shadow-sm">
          <CardContent className="p-6 space-y-4">
            <div className="grid grid-cols-2 md:grid-cols-4 gap-3 mb-4">
              <div className="rounded-xl bg-gray-50 border p-3">
                <p className="text-xs text-gray-500">本日の訪問数</p>
                <p className="text-lg font-bold">
                  {scopedRecords.filter((r) => r.date === recordDate).length}回
                </p>
              </div>

              <div className="rounded-xl bg-gray-50 border p-3">
                <p className="text-xs text-gray-500">合計時間</p>
                <p className="text-lg font-bold">
                  <TimeText
                    minutes={scopedRecords
                      .filter((r) => r.date === recordDate)
                      .reduce((sum, r) => sum + r.minutes, 0)}
                  />
                </p>
              </div>

              <div className="rounded-xl bg-gray-50 border p-3">
                <p className="text-xs text-gray-500">対応利用者数</p>
                <p className="text-lg font-bold">
                  {
                    new Set(
                      scopedRecords.filter((r) => r.date === recordDate).map((r) => r.user)
                    ).size
                  }
                  名
                </p>
              </div>

              <div className="rounded-xl bg-gray-50 border p-3">
                <p className="text-xs text-gray-500">平均対応時間</p>
                <p className="text-lg font-bold">
                  {scopedRecords.filter((r) => r.date === recordDate).length === 0
                    ? "0分"
                    : `${Math.round(
                        scopedRecords
                          .filter((r) => r.date === recordDate)
                          .reduce((sum, r) => sum + r.minutes, 0) /
                          scopedRecords.filter((r) => r.date === recordDate).length
                      )}分`}
                </p>
              </div>
            </div>

            <div className="flex flex-col md:flex-row md:items-center md:justify-between gap-3">
              <div>
                <h2 className="text-xl font-bold">実績確認</h2>
                <p className="text-sm text-gray-600">
                  「日別」「週別」「月間」「利用者別」の実績確認とCSV出力ができます。
                </p>
              </div>

              <select
                className="rounded-xl border p-3 bg-white"
                value={achievementUser}
                onChange={(e) => setAchievementUser(e.target.value)}
              >
                <option value="全員">全員</option>
                {scopedUsers.map((user) => (
                  <option key={user.id} value={user.name}>
                    {user.name}
                  </option>
                ))}
              </select>
            </div>

            <div className="grid grid-cols-4 gap-2">
              {["日別", "週別", "月間", "利用者別"].map((tab) => (
                <button
                  key={tab}
                  onClick={() => setAchievementTab(tab)}
                  className={`rounded-xl border p-3 font-bold ${
                    achievementTab === tab ? "bg-gray-900 text-white" : "bg-white"
                  }`}
                >
                  {tab}
                </button>
              ))}
            </div>

            {achievementTab === "日別" && (
              <div className="overflow-x-auto">
                <table className="w-full text-sm border-collapse">
                  <thead>
                    <tr className="bg-gray-100">
                      <th className="border p-2 text-left">日付</th>
                      <th className="border p-2 text-left">訪問回数</th>
                      <th className="border p-2 text-left">累積介入時間</th>
                    </tr>
                  </thead>

                  <tbody>
                    {Object.entries(achievementDaily).map(([date, s]) => (
                      <tr key={date}>
                        <td className="border p-2">{date}</td>
                        <td className="border p-2">{s.visits}回</td>
                        <td className="border p-2">
                          <TimeText minutes={s.minutes} />
                        </td>
                      </tr>
                    ))}
                  </tbody>
                </table>
              </div>
            )}

            {achievementTab === "週別" && (
              <div className="overflow-x-auto">
                <table className="w-full text-sm border-collapse">
                  <thead>
                    <tr className="bg-gray-100">
                      <th className="border p-2 text-left">週</th>
                      <th className="border p-2 text-left">訪問回数</th>
                      <th className="border p-2 text-left">累積介入時間</th>
                    </tr>
                  </thead>

                  <tbody>
                    {Object.entries(achievementWeekly).map(([week, s]) => (
                      <tr key={week}>
                        <td className="border p-2">{week}</td>
                        <td className="border p-2">{s.visits}回</td>
                        <td className="border p-2">
                          <TimeText minutes={s.minutes} />
                        </td>
                      </tr>
                    ))}
                  </tbody>
                </table>
              </div>
            )}

            {achievementTab === "月間" && (
              <div className="overflow-x-auto">
                <table className="w-full text-sm border-collapse">
                  <thead>
                    <tr className="bg-gray-100">
                      <th className="border p-2 text-left">月</th>
                      <th className="border p-2 text-left">訪問回数</th>
                      <th className="border p-2 text-left">累積介入時間</th>
                    </tr>
                  </thead>

                  <tbody>
                    {Object.entries(achievementMonthly).map(([month, s]) => (
                      <tr key={month}>
                        <td className="border p-2">{month}</td>
                        <td className="border p-2">{s.visits}回</td>
                        <td className="border p-2">
                          <TimeText minutes={s.minutes} />
                        </td>
                      </tr>
                    ))}
                  </tbody>
                </table>
              </div>
            )}

            {achievementTab === "利用者別" && (
              <div className="overflow-x-auto">
                <table className="w-full text-sm border-collapse">
                  <thead>
                    <tr className="bg-gray-100">
                      <th className="border p-2 text-left">利用者様</th>
                      <th className="border p-2 text-left">訪問回数</th>
                      <th className="border p-2 text-left">累積介入時間</th>
                    </tr>
                  </thead>

                  <tbody>
                    {Object.entries(achievementByUser).map(([user, s]) => (
                      <tr key={user}>
                        <td className="border p-2">{user}</td>
                        <td className="border p-2">{s.visits}回</td>
                        <td className="border p-2">
                          <TimeText minutes={s.minutes} />
                        </td>
                      </tr>
                    ))}
                  </tbody>
                </table>
              </div>
            )}
          </CardContent>
        </Card>
      </div>
    </div>
  );
}
