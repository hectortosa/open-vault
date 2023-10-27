<%*
const fileContent = tp.file.content;

const tableLines = fileContent.split('\n').filter(line => line.includes('|'));

const timeMarkRows = tableLines.slice(2).map(line => [line.split('|')[1].trim(), line.split('|')[2].trim()]);

const parseTimestamp = (str) => {
	const [datePart, timePart] = str.split(", ");
	const [day, month, year] = datePart.split(".");
	const [hour, minute] = timePart.split(".");
	return new Date(year, month - 1, day, hour, minute);
}

const result = timeMarkRows.reduce((acc, [project, timestampStr]) => {
    const timestamp = parseTimestamp(timestampStr);
    const found = acc.find(item => item.project === project);
    
    if (found) {
        // Calculate the difference from the last timestamp
        if (found.timestamps.length % 2 === 1) {  
            const lastTimestamp = parseTimestamp(found.timestamps[found.timestamps.length - 1]);
            const diff = (timestamp - lastTimestamp) / 1000;
            const diffHours = Math.floor(diff / 3600);
            const diffMinutes = Math.floor((diff % 3600) / 60);
            
            // Accumulate the sum of differences in totalMinutes
            found.totalMinutes += diffHours * 60 + diffMinutes;
        }
        found.timestamps.push(timestampStr);
    } else {
        acc.push({ project, timestamps: [timestampStr], totalMinutes: 0 });
    }
    return acc;
}, []);

result.forEach(item => {
    const hours = Math.floor(item.totalMinutes / 60);
    const minutes = item.totalMinutes % 60;
    item.sumOfDifferences = `${hours}h ${minutes}m`;
    // Remove totalMinutes property
    delete item.totalMinutes;
});

// Append the values to the current note
tR += result
	.map(mark => mark.project + ": **" + mark.sumOfDifferences + "**")
	.join('\n');
%>