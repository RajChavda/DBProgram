# DBProgram
Databse Files

// Make Global Object 

var obj_database = Database()


// ****************** DatabaseConfiguration
        
        obj_database = Database.share()
        obj_database.createEditableCopyOfDatabaseIfNeeded()
        
        
 // For Example this insert Query 
 
  let str_Sql_Insert = NSString(format: "insert into updateCartItem(product_id,product_name,product_variant,product_image_url,product_sell_price,product_list_price,product_discount,product_qty,product_max_qty,response) values ('%@','%@','%@','%@',%d,%d,%d,%d,%d,'%@')",str_variantId,str_name,str_variant_name,"",Int(str_SalePrice),Int(str_listPrice),Int(str_discout),Int(str_current),Int(maxQty),str_res) as String
  obj_database.insert(str_Sql_Insert)




//// For Decodable 


******************* Set Parameters 

struct Course: Decodable {
    let id: Int
    let name: String
    let imageUrl: String
    let link: String
    let number_of_lessons: Int
}


******************* API Called Methods 

func fetchApiMethods(strurl: String, completation: @escaping (Result<[Course], Error>) -> ()) {

    guard let url = URL(string: strurl) else { return }
    URLSession.shared.dataTask(with: url) { (data, res, err) in
        if let err = err {
            completation(.failure(err))
            return
        }
        do
        {
            let result = try JSONDecoder().decode([Course].self, from: data!)
            completation(.success(result))
        }
        catch let jsonError {
            completation(.failure(jsonError))
        }
        }.resume()
}


***************** Calling Methods 


fetchApiMethods(strurl: str_url) { (res) in
            switch res {
            case .success(let result):
                result.forEach({ (resa) in
                    print(resa)
                })
            case .failure(let err):
                print("Failed to load error", err)
            }
        }
        
        
        
***************** Api Methods 


func getApiCalled(_ str_eventName: String!, resolve: @escaping (_ name: AnyObject) -> Void)
    {
        let url1 = URL(string: str_eventName.addingPercentEncoding(withAllowedCharacters: .urlQueryAllowed)!)
        guard let url = url1 else {
            print("Error: cannot create URL")
            return
        }
        let urlRequest = URLRequest(url: url)
        // set up the session
        let config = URLSessionConfiguration.default
        let session = URLSession(configuration: config)
        let task = session.dataTask(with: urlRequest) { (data, response, error) in
            if let error = error {
                print("Api Error: \(error.localizedDescription)")
                return
            }
            guard let data = data, let response = response as? HTTPURLResponse else {
                print("Api Error: No response from API")
                return
            }
            int_HttpStatusCode = response.statusCode
            let object: NSDictionary?
            do {

                object = try JSONSerialization.jsonObject(with: data, options: .mutableContainers) as? NSDictionary
            } catch {
                object = nil
                print("Api Error")
                return
            }
            guard let json = object else {
                print("Api Parse Error")
                return
            }
            // Perform table updates on UI thread
            DispatchQueue.main.async {
                resolve(json)
            }
        }
        task.resume()
    }
  
  
  func PostApiCalled(_ str_eventName: String, parameter: AnyObject , headers: HTTPHeaders, resolve:@escaping (_ name: AnyObject) -> Void)
    {
        let str_url = URL.init(string: str_eventName)
        Alamofire.request(str_url!, method: .post, parameters: parameter as? Parameters, encoding: JSONEncoding.default, headers: headers).responseJSON { (responseData) -> Void in
            let httpStatusCode = responseData.response?.statusCode
            int_HttpStatusCode = httpStatusCode!
            if responseData.result.value != nil
            {
                int_HttpStatusCode = httpStatusCode!
                resolve(responseData.result.value! as AnyObject)
            }
            else
            {
                print(responseData)
                resolve(["errorMessage":str_serverError] as AnyObject)
            }
        }
    }
