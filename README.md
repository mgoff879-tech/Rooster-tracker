# Rooster-tracker
Flesh score calculator 
import React, { useState, useEffect } from ‘react’;
import { Plus, Trash2, BarChart3 } from ‘lucide-react’;

const SCORE_VALUES = {
‘-’: 3.0,
‘3’: 3.0,
‘+’: 3.5,
‘4’: 4.0
};

const ScoreTracker = () => {
const [barns, setBarns] = useState([]);
const [newBarnName, setNewBarnName] = useState(’’);
const [loading, setLoading] = useState(true);

useEffect(() => {
loadBarns();
}, []);

const loadBarns = async () => {
try {
const result = await window.storage.list(‘barn:’);
if (result && result.keys) {
const barnData = await Promise.all(
result.keys.map(async (key) => {
try {
const data = await window.storage.get(key);
return data ? JSON.parse(data.value) : null;
} catch {
return null;
}
})
);
setBarns(barnData.filter(b => b !== null).sort((a, b) => b.timestamp - a.timestamp));
}
} catch (error) {
console.log(‘No existing barns found’);
} finally {
setLoading(false);
}
};

const saveBarn = async (barn) => {
try {
await window.storage.set(`barn:${barn.id}`, JSON.stringify(barn));
} catch (error) {
console.error(‘Error saving barn:’, error);
}
};

const addBarn = () => {
if (!newBarnName.trim()) return;

```
const newBarn = {
  id: Date.now().toString(),
  name: newBarnName.trim(),
  scores: { '-': 0, '3': 0, '+': 0, '4': 0 },
  timestamp: Date.now()
};

saveBarn(newBarn);
setBarns([newBarn, ...barns]);
setNewBarnName('');
```

};

const updateScore = (barnId, scoreType) => {
const updatedBarns = barns.map(barn => {
if (barn.id === barnId) {
const updated = {
…barn,
scores: { …barn.scores, [scoreType]: barn.scores[scoreType] + 1 },
timestamp: Date.now()
};
saveBarn(updated);
return updated;
}
return barn;
});
setBarns(updatedBarns.sort((a, b) => b.timestamp - a.timestamp));
};

const deleteBarn = async (barnId) => {
try {
await window.storage.delete(`barn:${barnId}`);
setBarns(barns.filter(b => b.id !== barnId));
} catch (error) {
console.error(‘Error deleting barn:’, error);
}
};

const calculateStats = (scores) => {
const counts = scores;
const sums = {
‘-’: counts[’-’] * SCORE_VALUES[’-’],
‘3’: counts[‘3’] * SCORE_VALUES[‘3’],
‘+’: counts[’+’] * SCORE_VALUES[’+’],
‘4’: counts[‘4’] * SCORE_VALUES[‘4’]
};

```
const totalRoosters = Object.values(counts).reduce((a, b) => a + b, 0);
const totalScore = Object.values(sums).reduce((a, b) => a + b, 0);
const average = totalRoosters > 0 ? (totalScore / totalRoosters).toFixed(2) : '0.00';

const percentages = {
  '-': totalRoosters > 0 ? ((counts['-'] / totalRoosters) * 100).toFixed(2) : '0.00',
  '3': totalRoosters > 0 ? ((counts['3'] / totalRoosters) * 100).toFixed(2) : '0.00',
  '+': totalRoosters > 0 ? ((counts['+'] / totalRoosters) * 100).toFixed(2) : '0.00',
  '4': totalRoosters > 0 ? ((counts['4'] / totalRoosters) * 100).toFixed(2) : '0.00'
};

return { counts, sums, totalRoosters, totalScore, average, percentages };
```

};

if (loading) {
return <div className="min-h-screen bg-gray-50 flex items-center justify-center">
<p className="text-gray-600">Loading…</p>
</div>;
}

return (
<div className="min-h-screen bg-gray-50 p-4">
<div className="max-w-6xl mx-auto">
<div className="bg-white rounded-lg shadow-md p-6 mb-6">
<h1 className="text-3xl font-bold text-gray-800 mb-6 flex items-center gap-2">
<BarChart3 className="w-8 h-8" />
Rooster Fleshing Score Tracker
</h1>

```
      <div className="flex gap-2 mb-4">
        <input
          type="text"
          value={newBarnName}
          onChange={(e) => setNewBarnName(e.target.value)}
          onKeyPress={(e) => e.key === 'Enter' && addBarn()}
          placeholder="Enter barn name..."
          className="flex-1 px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
        />
        <button
          onClick={addBarn}
          className="bg-blue-600 text-white px-6 py-2 rounded-lg hover:bg-blue-700 transition flex items-center gap-2"
        >
          <Plus className="w-5 h-5" />
          Add Barn
        </button>
      </div>
    </div>

    {barns.length === 0 ? (
      <div className="bg-white rounded-lg shadow-md p-12 text-center">
        <p className="text-gray-500 text-lg">No barns yet. Add your first barn to get started!</p>
      </div>
    ) : (
      <div className="space-y-6">
        {barns.map(barn => {
          const stats = calculateStats(barn.scores);
          
          return (
            <div key={barn.id} className="bg-white rounded-lg shadow-md p-6">
              <div className="flex justify-between items-center mb-4">
                <h2 className="text-2xl font-bold text-gray-800">{barn.name}</h2>
                <button
                  onClick={() => deleteBarn(barn.id)}
                  className="text-red-600 hover:text-red-700 p-2 hover:bg-red-50 rounded-lg transition"
                >
                  <Trash2 className="w-5 h-5" />
                </button>
              </div>

              <div className="grid grid-cols-4 gap-3 mb-6">
                {Object.keys(SCORE_VALUES).map(scoreType => (
                  <button
                    key={scoreType}
                    onClick={() => updateScore(barn.id, scoreType)}
                    className="bg-gradient-to-br from-blue-500 to-blue-600 text-white p-6 rounded-lg hover:from-blue-600 hover:to-blue-700 transition shadow-md hover:shadow-lg transform hover:scale-105"
                  >
                    <div className="text-4xl font-bold mb-2">{scoreType}</div>
                    <div className="text-sm opacity-90">Score: {SCORE_VALUES[scoreType]}</div>
                    <div className="text-2xl font-semibold mt-2">{barn.scores[scoreType]}</div>
                  </button>
                ))}
              </div>

              <div className="overflow-x-auto">
                <table className="w-full border-collapse border-2 border-gray-300">
                  <tbody>
                    <tr className="bg-blue-50">
                      <td className="border border-gray-300 px-4 py-2 font-semibold text-center">{stats.sums['-'].toFixed(1)}</td>
                      <td className="border border-gray-300 px-4 py-2 font-semibold text-center">{stats.sums['3'].toFixed(1)}</td>
                      <td className="border border-gray-300 px-4 py-2 font-semibold text-center">{stats.sums['+'].toFixed(1)}</td>
                      <td className="border border-gray-300 px-4 py-2 font-semibold text-center">{stats.sums['4'].toFixed(1)}</td>
                      <td className="border border-gray-300 px-4 py-2 font-semibold text-center bg-yellow-100">{stats.totalScore.toFixed(1)}</td>
                      <td className="border border-gray-300 px-4 py-2 font-semibold text-center bg-green-100">{stats.average}</td>
                    </tr>
                    <tr className="bg-gray-100">
                      <td className="border border-gray-300 px-4 py-2 text-center font-bold text-lg">-</td>
                      <td className="border border-gray-300 px-4 py-2 text-center font-bold text-lg">3</td>
                      <td className="border border-gray-300 px-4 py-2 text-center font-bold text-lg">+</td>
                      <td className="border border-gray-300 px-4 py-2 text-center font-bold text-lg">4</td>
                      <td className="border border-gray-300 px-4 py-2 text-center font-bold bg-yellow-50">Total</td>
                      <td className="border border-gray-300 px-4 py-2 text-center font-bold bg-green-50">Avg</td>
                    </tr>
                    <tr>
                      <td className="border border-gray-300 px-4 py-2 text-center text-lg">{stats.counts['-']}</td>
                      <td className="border border-gray-300 px-4 py-2 text-center text-lg">{stats.counts['3']}</td>
                      <td className="border border-gray-300 px-4 py-2 text-center text-lg">{stats.counts['+']}</td>
                      <td className="border border-gray-300 px-4 py-2 text-center text-lg">{stats.counts['4']}</td>
                      <td className="border border-gray-300 px-4 py-2 text-center font-bold text-lg bg-yellow-50">{stats.totalRoosters}</td>
                      <td className="border border-gray-300 px-4 py-2 bg-gray-50"></td>
                    </tr>
                    <tr className="bg-purple-50">
                      <td className="border border-gray-300 px-4 py-2 text-center">{stats.percentages['-']}%</td>
                      <td className="border border-gray-300 px-4 py-2 text-center">{stats.percentages['3']}%</td>
                      <td className="border border-gray-300 px-4 py-2 text-center">{stats.percentages['+']}%</td>
                      <td className="border border-gray-300 px-4 py-2 text-center">{stats.percentages['4']}%</td>
                      <td className="border border-gray-300 px-4 py-2 text-center font-bold bg-yellow-100">100%</td>
                      <td className="border border-gray-300 px-4 py-2 bg-gray-50"></td>
                    </tr>
                  </tbody>
                </table>
              </div>
            </div>
          );
        })}
      </div>
    )}
  </div>
</div>
```

);
};

export default ScoreTracker;
