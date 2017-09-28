package com.example.weatherlocation;

/*
 * Copyright (C) 2012 The Android Open Source Project
 *
 * Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except
 * in compliance with the License. You may obtain a copy of the License at
 *
 * http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software distributed under the License
 * is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express
 * or implied. See the License for the specific language governing permissions and limitations under
 * the License.
 */

import org.xmlpull.v1.XmlPullParser;
import org.xmlpull.v1.XmlPullParserException;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

public class XMLParser {

	public List<Stations> parse(XmlPullParser parser)
			throws XmlPullParserException, IOException {
		try {
			return readStations(parser);
		} finally {

		}
	}

	private List<Stations> readStations(XmlPullParser parser)
			throws XmlPullParserException, IOException {
		List<Stations> entries = new ArrayList<Stations>();
		int tag = parser.next();
		while (tag != XmlPullParser.END_DOCUMENT) {
			if (parser.getEventType() != XmlPullParser.START_TAG) {
				tag = parser.next();
				continue;
			}
			String name = parser.getName();
			if (name.equals("station")) {
				entries.add(readStation(parser));
			}
			tag = parser.next();

		}
		return entries;
	}

	public static class Stations {
		public final String link;
		public final String latitude;
		public final String longitude;

		private Stations(String link, String latitude, String longitude) {
			this.link = link;
			this.latitude = latitude;
			this.longitude = longitude;
		}
	}

	private Stations readStation(XmlPullParser parser)
			throws XmlPullParserException, IOException {
		String link = null;
		String latitude = null;
		String longitude = null;
		while (parser.next() != XmlPullParser.END_TAG) {
			if (parser.getEventType() != XmlPullParser.START_TAG) {
				continue;
			}
			String name = parser.getName();
			if (name.equals("xml_url")) {
				link = readLink(parser);
			} else if (name.equals("latitude")) {
				latitude = readLongitude(parser);
			} else if (name.equals("longitude")) {
				longitude = readLatitude(parser);
			} else {
				skip(parser);
			}
		}
		return new Stations(link, latitude, longitude);
	}

	private String readLink(XmlPullParser parser) throws IOException,
			XmlPullParserException {
		String link = readText(parser);
		return link;
	}

	private String readLatitude(XmlPullParser parser) throws IOException,
			XmlPullParserException {
		String latitude = readText(parser);
		return latitude;
	}

	private String readLongitude(XmlPullParser parser) throws IOException,
			XmlPullParserException {
		String longitude = readText(parser);
		return longitude;
	}
	
	private String readText(XmlPullParser parser) throws IOException,
			XmlPullParserException {
		String result = "";
		if (parser.next() == XmlPullParser.TEXT) {
			result = parser.getText();
			parser.nextTag();
		}
		return result;
	}
	
	private void skip(XmlPullParser parser) throws XmlPullParserException,
			IOException {
		if (parser.getEventType() != XmlPullParser.START_TAG) {
			throw new IllegalStateException();
		}
		int depth = 1;
		while (depth != 0) {
			switch (parser.next()) {
			case XmlPullParser.END_TAG:
				depth--;
				break;
			case XmlPullParser.START_TAG:
				depth++;
				break;
			}
		}
	}
}
