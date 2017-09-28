package com.example.weatherlocation;

import java.io.IOException;
import java.io.InputStream;
import java.io.StringReader;
import java.net.HttpURLConnection;
import java.net.MalformedURLException;
import java.net.URL;
import java.util.List;

import javax.xml.parsers.ParserConfigurationException;
import javax.xml.parsers.SAXParser;
import javax.xml.parsers.SAXParserFactory;

import org.xml.sax.Attributes;
import org.xml.sax.InputSource;
import org.xml.sax.SAXException;
import org.xml.sax.helpers.DefaultHandler;
import org.xmlpull.v1.XmlPullParserException;

import com.example.weatherlocation.XMLParser.Stations;

import android.app.Activity;
import android.app.AlertDialog;
import android.app.ProgressDialog;
import android.content.res.XmlResourceParser;
import android.location.Location;
import android.location.LocationListener;
import android.location.LocationManager;
import android.os.AsyncTask;
import android.os.Bundle;
import android.util.Log;
import android.view.Menu;
import android.view.MenuItem;
import android.widget.TextView;

public class MainActivity extends Activity implements LocationListener {

	String stringURL;
	TextView station, stationData, observation, observationData, weather,
		weatherData, temperature, temperatureData, wind, windData;
	
	LocationManager lm;
	double latitude;
	double longitude;
	List<Stations> stations = null;
	AsyncTask<Void, Integer, Void> parseStations;
	AsyncTask<Void, Integer, Void> downloadXML;
	XmlResourceParser parser;
	String idString, observationString, weatherString, temperatureString, windString;
	ProgressDialog pd;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);

		initTextViews();
		pd = new ProgressDialog(MainActivity.this);
		pd.setMessage("Getting Weather Data...");
		pd.setProgressStyle(pd.STYLE_SPINNER);

		lm = (LocationManager) getSystemService(LOCATION_SERVICE);
		Log.i("",
				"Enable: " + lm.isProviderEnabled(LocationManager.GPS_PROVIDER));

		if (lm.isProviderEnabled(LocationManager.GPS_PROVIDER)) {
			if (lm.getLastKnownLocation(LocationManager.GPS_PROVIDER) != null) {
				latitude = lm
						.getLastKnownLocation(LocationManager.GPS_PROVIDER)
						.getLatitude();
				longitude = lm.getLastKnownLocation(
						LocationManager.GPS_PROVIDER).getLongitude();
			}
			lm.requestLocationUpdates(LocationManager.GPS_PROVIDER, 5000, 0,
					this);
		} else {
			new AlertDialog.Builder(this).setMessage("Enable GPS!").show();
		}

		Log.i("", "Lat: " + latitude);
		Log.i("", "Long:" + longitude);

		parseStations = new AsyncTask<Void, Integer, Void>() {

			@Override
			protected Void doInBackground(Void... params) {

				parser = getResources().getXml(R.xml.station_lookup2);
				XMLParser xmlParser = new XMLParser();

				try {
					Log.i("", "readStation   " + parser.getName());
					stations = xmlParser.parse(parser);

				} catch (XmlPullParserException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				} catch (IOException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}

				return null;
			}

			@Override
			protected void onPreExecute() {
				pd.show();
				super.onPreExecute();
			}

			@Override
			protected void onPostExecute(Void result) {
				double minDistance = calcDistance(longitude, latitude,
						Double.parseDouble(stations.get(0).longitude),
						Double.parseDouble(stations.get(0).latitude));
				stringURL = stations.get(0).link;
				double distance;

				for (int i = 0; i < stations.size(); i++) {
					distance = calcDistance(longitude, latitude,
							Double.parseDouble(stations.get(i).longitude),
							Double.parseDouble(stations.get(i).latitude));
					if (distance < minDistance) {
						minDistance = distance;
						stringURL = stations.get(i).link;
					}

				}
				Log.i("", "url:   " + stringURL);
				Log.i("", "distance:   " + minDistance);
				downloadXML.execute();
				super.onPostExecute(result);
			}

		};

		parseStations.execute();
		downloadXML = new AsyncTask<Void, Integer, Void>() {

			@Override
			protected Void doInBackground(Void... params) {

				try {
					URL url = new URL(stringURL);
					HttpURLConnection httpConnection = (HttpURLConnection) url
							.openConnection();
					httpConnection.setRequestMethod("GET");
					httpConnection.connect();
					httpConnection.getContentType();
					Log.i("", "Resposne: " + httpConnection.getResponseCode());
					InputStream is = httpConnection.getInputStream();
					String receivedData = new String();
					int nread = 0;
					byte[] bytes = new byte[512];
					while ((nread = is.read(bytes)) > 0) {
						receivedData = receivedData.concat(new String(bytes, 0,
								nread));

					}

					parseDataString(receivedData);
				} catch (MalformedURLException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				} catch (IOException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
				return null;

			}

			@Override
			protected void onPostExecute(Void result) {

				stationData.setText(idString);
				observationData.setText(observationString);
				weatherData.setText(weatherString);
				temperatureData.setText(temperatureString);
				windData.setText(windString);
				pd.hide();

				super.onPostExecute(result);
			}

		};

	}
	
	public void initTextViews() {
		station = (TextView) findViewById(R.id.station);
		stationData = (TextView) findViewById(R.id.stationdata);
		observation = (TextView) findViewById(R.id.observation);
		observationData = (TextView) findViewById(R.id.observationdata);
		weather = (TextView) findViewById(R.id.weather);
		weatherData = (TextView) findViewById(R.id.weatherdata);
		temperature = (TextView) findViewById(R.id.tempurature);
		temperatureData = (TextView) findViewById(R.id.tempuraturedata);
		wind = (TextView) findViewById(R.id.wind);
		windData = (TextView) findViewById(R.id.winddata);
	}

	public void parseDataString(String dataString) {
		try {
			SAXParser parser = SAXParserFactory.newInstance().newSAXParser();
			try {
				parser.parse(new InputSource(new StringReader(dataString)),
						new DefaultHandler() {

							boolean temperature = false;
							boolean id = false;
							boolean time = false;
							boolean weather = false;
							boolean wind = false;

							@Override
							public void startDocument() throws SAXException {
								// TODO Auto-generated method stub
								super.startDocument();
								// Log.i("","Start of document");
							}

							@Override
							public void startElement(String uri,
									String localName, String qName,
									Attributes attributes) throws SAXException {
								// Log.i("", "Start of element: " + qName +
								// " , " + localName);

								for (int i = 0; i < attributes.getLength(); ++i) {
									// Log.i("", "Attr: " +
									// attributes.getQName(i) + ", " +
									// attributes.getLocalName(i) + " Val: " +
									// attributes.getValue(i));
								}

								if (qName.equals("station_id")) {
									id = true;
								}
								if (qName.equals("observation_time")) {
									time = true;
								}
								if (qName.equals("weather")) {
									weather = true;
								}
								if (qName.equals("temperature_string")) {
									temperature = true;
								}
								if (qName.equals("wind_string")) {
									wind = true;
								}

							}

							@Override
							public void endElement(String uri,
									String localName, String qName)
									throws SAXException {
								// TODO Auto-generated method stub
								super.endElement(uri, localName, qName);
								// Log.i("", "End of element: " + qName + " , "
								// + localName);
								if (qName.equals("station_id")) {
									id = false;
								}
								if (qName.equals("observation_time")) {
									time = false;
								}
								if (qName.equals("weather")) {
									weather = false;
								}
								if (qName.equals("temperature_string")) {
									temperature = false;
								}
								if (qName.equals("wind_string")) {
									wind = false;
								}

							}

							@Override
							public void endDocument() throws SAXException {
								// TODO Auto-generated method stub
								super.endDocument();
								// Log.i("", "End of document");
							}

							@Override
							public void characters(char[] ch, int start,
									int length) throws SAXException {

								if (id) {
									idString = new String(ch, start, length);
								}
								if (time) {
									observationString = new String(ch, start, length);
								}
								if (weather) {
									weatherString = new String(ch, start,
											length);
								}
								if (temperature) {
									temperatureString = new String(ch, start,
											length);
								}
								if (wind) {
									windString = new String(ch, start, length);
								}

							}

						});
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}

		} catch (ParserConfigurationException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (SAXException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}

	private double calcDistance(double rLong1, double rLat1, double rLong2,
			double rLat2) {

		rLong1 = Math.toRadians(rLong1);
		rLong2 = Math.toRadians(rLong2);
		rLat1 = Math.toRadians(rLat1);
		rLat2 = Math.toRadians(rLat2);

		double dist = 0;
		double dLong = rLong2 - rLong1;
		double dLat = rLat2 - rLat1;
		double a = Math.sin(dLat / 2d) * Math.sin(dLat / 2d)
				+ Math.sin(dLong / 2d) * Math.sin(dLong / 2d) * Math.cos(rLat1)
				* Math.cos(rLat2);
		double c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
		dist = (6371 * c) * 0.621371d;

		return dist;
	}

	@Override
	public boolean onCreateOptionsMenu(Menu menu) {
		// Inflate the menu; this adds items to the action bar if it is present.
		getMenuInflater().inflate(R.menu.main, menu);
		return true;
	}

	@Override
	public boolean onOptionsItemSelected(MenuItem item) {
		// Handle action bar item clicks here. The action bar will
		// automatically handle clicks on the Home/Up button, so long
		// as you specify a parent activity in AndroidManifest.xml.
		int id = item.getItemId();
		if (id == R.id.action_settings) {
			return true;
		}
		return super.onOptionsItemSelected(item);
	}

	@Override
	public void onLocationChanged(Location location) {
		// TODO Auto-generated method stub

	}

	@Override
	public void onProviderDisabled(String provider) {
		// TODO Auto-generated method stub

	}

	@Override
	public void onProviderEnabled(String provider) {
		// TODO Auto-generated method stub

	}

	@Override
	public void onStatusChanged(String provider, int status, Bundle extras) {
		// TODO Auto-generated method stub

	}
}
