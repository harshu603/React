npx create-react-app kml-viewer
cd kml-viewer
npm install react-leaflet leaflet xml2js

import React, { useState } from 'react';
import { MapContainer, TileLayer, Polyline } from 'react-leaflet';
import { parseStringPromise } from 'xml2js';
import 'leaflet/dist/leaflet.css';
import L from 'leaflet';

const KmlViewer = () => {
  const [kmlData, setKmlData] = useState(null);
  const [elementCount, setElementCount] = useState({});
  const [detailedData, setDetailedData] = useState([]);
  const [mapLines, setMapLines] = useState([]);

  const handleFileChange = async (event) => {
    const file = event.target.files[0];
    if (file) {
      const text = await file.text();
      parseKml(text);
    }
  };

  const parseKml = async (kmlText) => {
    const result = await parseStringPromise(kmlText);
    const placemarks = result.kml.Document[0].Placemark || [];
    const counts = {};
    const lines = [];

    placemarks.forEach((placemark) => {
      const geometry = placemark.LineString || placemark.MultiGeometry || [];
      if (geometry.length) {
        const coordinates = geometry[0].coordinates[0].trim().split(' ').map(coord => {
          const [lng, lat] = coord.split(',').map(Number);
          return [lat, lng];
        });
        lines.push(coordinates);
        counts['LineString'] = (counts['LineString'] || 0) + 1;
      }
    });

    setKmlData(result);
    setElementCount(counts);
    setMapLines(lines);
  };

  const handleSummary = () => {
    alert(JSON.stringify(elementCount, null, 2));
  };

  const handleDetailed = () => {
    const details = mapLines.map((line, index) => ({
      type: 'LineString',
      length: calculateLength(line),
    }));
    setDetailedData(details);
    alert(JSON.stringify(details, null, 2));
  };

  const calculateLength = (line) => {
    let totalLength = 0;
    for (let i = 0; i < line.length - 1; i++) {
      const [lat1, lng1] = line[i];
      const [lat2, lng2] = line[i + 1];
      totalLength += getDistanceFromLatLonInKm(lat1, lng1, lat2, lng2);
    }
    return totalLength;
  };

  const getDistanceFromLatLonInKm = (lat1, lon1, lat2, lon2) => {
    const R = 6371; // Radius of the earth in km
    const dLat = deg2rad(lat2 - lat1);
    const dLon = deg2rad(lon2 - lon1);
    const a =
      Math.sin(dLat / 2) * Math.sin(dLat / 2) +
      Math.cos(deg2rad(lat1)) * Math.cos(deg2rad(lat2)) *
      Math.sin(dLon / 2) * Math.sin(dLon / 2);
    const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
    return R * c; // Distance in km
  };

  const deg2rad = (deg) => {
    return deg * (Math.PI / 180);
  };

  return (
    <div>
      <input type="file" accept=".kml" onChange={handleFileChange} />
      <button onClick={handleSummary}>Summary</button>
      <button onClick={handleDetailed}>Detailed</button>
      <MapContainer center={[0, 0]} zoom={2} style={{ height: '500px', width: '100%' }}>
        <TileLayer
          url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
          attribution='&copy; <a href="https://www.open
