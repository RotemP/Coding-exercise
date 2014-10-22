Coding-exercise
===============

import java.io.BufferedReader;
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.IOException;
import java.io.FileWriter;
import java.util.*;

public class WaterNetwork {
	
	private static HashMap<String, ArrayList<String>> parentChildren= new HashMap<String, ArrayList<String>>();
	private static HashMap<String, int[]> allZonesValues= new HashMap<String, int[]>();
	private static String[] dates = new String[8];
	
	public static HashMap<String, ArrayList<String>> getParentChildren(){
		return parentChildren;
	}
	public static HashMap<String, int[]> getAllZonesValues(){
		return allZonesValues;
	}
	public static void setParentChildren(){
		parentChildren= new HashMap<String, ArrayList<String>>();
	}
	public static void setAllZonesValues(){
		allZonesValues= new HashMap<String, int[]>();
	}
	public static ArrayList<String[]> readCsvFile(String csvFile){
		ArrayList<String[]> stringsList = new ArrayList<String[]>();
		BufferedReader br = null;
		String line = "";
		String cvsSplitBy = ",";
		try {
			br = new BufferedReader(new FileReader(csvFile));
			while ((line = br.readLine()) != null) {
			    // use comma as separator
				String[] supplyValues = line.split(cvsSplitBy);
				stringsList.add(supplyValues);
			}
		} catch (FileNotFoundException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		} finally {
			if (br != null) {
				try {
					br.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
		}
		return stringsList;
	}	
	
	public static void parseZones(ArrayList<String[]> zonesList){
		//ignore first line
		for (int i=1; i<zonesList.size();i++){
			String[] zones=zonesList.get(i);
			String zoneName = zones[0];
			//add new zone to hash map
			if (!allZonesValues.containsKey(zoneName)){
				int[] values = new int[8];
				allZonesValues.put(zoneName, values);
			}
			//add zone to his parent zone
			if (zones.length>1){
				String parentName = zones[1];
				//add new zone parent to hash map
				if (!allZonesValues.containsKey(parentName)){
					int[] values = new int[8];
					allZonesValues.put(parentName, values);
				}
				if (parentChildren.containsKey(parentName)){
					ArrayList<String> childrens = parentChildren.get(parentName);
					if (!childrens.contains(zoneName)){
						childrens.add(zoneName);
						parentChildren.put(parentName, childrens);
					}
				}
				else{
					ArrayList<String> childrens = new ArrayList<String>();
					childrens.add(zoneName);
					parentChildren.put(parentName, childrens);
				}
			}
		}
	}
	
	public static void parseValues(ArrayList<String[]> valuesList){
		//ignore first line
		for (int i=1; i<valuesList.size();i++){
			String[] supplyValues=valuesList.get(i);
			String supplyZone = supplyValues[0];
			String date	= supplyValues[1];
			int slashIndex = date.indexOf('/');
			String shortDate = date.substring(0, slashIndex);
			int dateNum = Integer.parseInt(shortDate);
			if (dates[dateNum]==null){
				dates[dateNum]=date;
			}
			String supply = supplyValues[2];
			//if doesn't exist, add new zone to hash map
			if (!allZonesValues.containsKey(supplyZone)){
				int[] values = new int[8];
				allZonesValues.put(supplyZone, values);
			}
			//update the hash map values in the relevant date
			int[] values = allZonesValues.get(supplyZone);
			values[dateNum]=Integer.parseInt(supply);
			allZonesValues.put(supplyZone, values);
		}
	}
	
	public static int getChildrenValue(ArrayList<String> childrens, int date){
		int sum = 0;
		int size = childrens.size();
		for (int i=0; i<size; i++){
			String child_name = childrens.get(i);
			int[] values = allZonesValues.get(child_name);
			if (values[date]!=0){
				sum+=values[date];
			}
			//one of the direct children has no value at the given date 
			else{
				return 0;
			}
		}
		return sum;
	}
	public static void main(String[] args) {
		//String curDir=System.getProperty("user.dir");
		ArrayList<String[]> zonesList = readCsvFile("Supply zones.csv");
		parseZones(zonesList);
		ArrayList<String[]> valuesList = readCsvFile("Supply values.csv");
		parseValues(valuesList);
		String outputFile = "Zones output.csv";
		try
		{
		    FileWriter writer = new FileWriter(outputFile);
		    writer.append("zone name,date,value,type");
		    writer.append('\n');
			//print for each zone supply its values per date
		    String type="";
		    String value="";
		    boolean appendLine;
			for (String zone : allZonesValues.keySet()) {
			    int[] values = allZonesValues.get(zone);
				for (int dateNum=1; dateNum<8; dateNum++) {
					appendLine=false;
					 if (values[dateNum]!=0){
						 System.out.println(" zone name:" + zone+", date:"+dates[dateNum]+", value:"+values[dateNum]+", actual");
						 type = "actual";
						 value = "" + values[dateNum];
						 appendLine=true;
				    }
			    	else{
			    		//calculate values sum of zone children
			    		//sum zone direct children values for a certain date 
			    		if (parentChildren.containsKey(zone)){
			    			ArrayList<String> childrens = parentChildren.get(zone);
				    		int childrenVal = getChildrenValue(childrens, dateNum);
				    		if (childrenVal!=0){
				    			System.out.println(" zone name:" + zone+", date:"+dates[dateNum]+", value:"+childrenVal+", type:aggregated");
							    type = "aggregated";
								value = "" + childrenVal;
								appendLine=true;
				    		}
			    		}
			    	}
					 if (appendLine){
					    writer.append(zone);
					    writer.append(',');
					    writer.append(dates[dateNum]);
					    writer.append(',');
					    writer.append(value);
					    writer.append(',');
					    writer.append(type);
					    writer.append('\n');
					 }
				}
			}
		    writer.flush();
		    writer.close();
		}
		catch(IOException e)
		{
		     e.printStackTrace();
		} 
    return;
	}
}


import static org.junit.Assert.*;

import java.util.ArrayList;
import java.util.HashMap;
import org.junit.Test;


public class WaterNetworkTest {

	@Test
	public void testReadCsvFile() {
		WaterNetwork.setParentChildren();
		String csvFile="test read.csv";
		ArrayList<String[]> stringList = WaterNetwork.readCsvFile(csvFile);
		assertEquals(stringList.size(),12);
		String[] zones=stringList.get(0);
		assertEquals(zones[0],"zone name");
		zones=stringList.get(1);
		assertEquals(zones.length,1);
		zones=stringList.get(2);
		assertEquals(zones[1],"Gush Dan");
		zones=stringList.get(8);
		//System.out.println(" zones[2]:" + zones[2]);
		assertEquals(zones[2],"9");
	}

	@Test
	public void testParseZones1() {	
		WaterNetwork.setParentChildren();
		ArrayList<String[]> zonesList = new ArrayList<String[]>();
		String[] line1 = {"zone name", "parent zone name"};
		zonesList.add(line1);
		String[] line2 = {"Gush Dan"};
		zonesList.add(line2);
		String[] line3 = {"Tel Aviv"};
		zonesList.add(line3);
		WaterNetwork.parseZones(zonesList);
		HashMap<String, ArrayList<String>> parentChildren = WaterNetwork.getParentChildren();
		assertTrue(parentChildren.isEmpty());
	}
	
	@Test
	public void testParseZones2() {
		ArrayList<String[]> zonesList = new ArrayList<String[]>();
		String[] line1 = {"zone name", "parent zone name"};
		zonesList.add(line1);
		String[] line2 = {"Ramat Aviv", "Tel Aviv"};
		zonesList.add(line2);
		String[] line3 = {"Hatikva", "Tel Aviv"};
		zonesList.add(line3);
		zonesList.add(line3);
		String[] line4 = {"Amidar",	"Ramat Gan"};
		zonesList.add(line4);
		WaterNetwork.parseZones(zonesList);
		HashMap<String, ArrayList<String>> parentChildren = WaterNetwork.getParentChildren();
		assertEquals(parentChildren.size(),2);
		assertTrue(parentChildren.containsKey("Tel Aviv"));
		ArrayList<String> childrens = parentChildren.get("Tel Aviv");
		assertEquals(childrens.get(0),"Ramat Aviv");
		assertEquals(childrens.get(1),"Hatikva");
		childrens = parentChildren.get("Ramat Gan");
		assertEquals(childrens.get(0),"Amidar");
	}
	@Test
	public void testParseValues() {
		WaterNetwork.setParentChildren();
		ArrayList<String[]> zonesList = new ArrayList<String[]>();
		String[] line = {"zone name", "parent zone name"};
		zonesList.add(line);
		String[] line1 = {"Tel Aviv", "Gush Dan"};
		zonesList.add(line1);
		String[] line2 = {"Hatikva", "Tel Aviv"};
		zonesList.add(line2);
		String[] line3 = {"Ramat Aviv", "Tel Aviv"};
		zonesList.add(line3);
		WaterNetwork.parseZones(zonesList);
		ArrayList<String[]> valuesList = new ArrayList<String[]>();
		String[] line4 = {"supply", "date", "supply (Ml/d)"};
		valuesList.add(line4);
		String[] line5 = {"Tel Aviv", "01/01/2014", "10"};
		valuesList.add(line5);
		String[] line6 = {"Tel Aviv", "06/01/2014", "9"};
		valuesList.add(line6);
		String[] line7 = {"Ramat Aviv", "02/01/2014", "2"};
		valuesList.add(line7);
		String[] line8 = {"Hatikva", "02/01/2014", "3"};
		valuesList.add(line8);
		String[] line9 = {"Hatikva", "03/01/2014", "8"};
		valuesList.add(line9);
		String[] line10 = {"Hatikva", "06/01/2014", "4"};
		valuesList.add(line10);
		WaterNetwork.parseValues(valuesList);
		HashMap<String, int[]> allZonesValues = WaterNetwork.getAllZonesValues();
		int[] values = allZonesValues.get("Gush Dan");
		System.out.println("values[1] of gush dan: " + values[1]);
		assertEquals(values[1],0);
		values = allZonesValues.get("Ramat Aviv");
		assertEquals(values[2],2);
		values = allZonesValues.get("Hatikva");
		assertEquals(values[6],4);
	}

	@Test
	public void testGetChildrenValue() {
		testParseValues();
		HashMap<String, ArrayList<String>> parentChildren = WaterNetwork.getParentChildren();
		ArrayList <String> children = parentChildren.get("Tel Aviv");
		//all direct children have a value on 02/01
		int val = WaterNetwork.getChildrenValue(children, 2);
		assertEquals(val,5);
		//one direct child has no value on 03/01
		val = WaterNetwork.getChildrenValue(children, 3);
		assertEquals(val,0);
		//all direct children (only Tel Aviv) have a value on 06/01
		children = parentChildren.get("Gush Dan");
		val = WaterNetwork.getChildrenValue(children, 6);
		assertEquals(val,9);
	}

}


